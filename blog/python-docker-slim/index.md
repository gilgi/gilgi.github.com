.. title: Building slim Docker images for Python with multi-stage builds
.. slug: python-docker-slim
.. date: 2020-05-23
.. tags: Python, Docker
.. has_math: false
.. category:
.. description:
.. type: text

It seems like these days everyone has their own bespoke recommended method for Dockerizing Python applications.
These include [several](https://hub.docker.com/r/jupyter/datascience-notebook/) [base](https://hub.docker.com/r/wiseio/datascience-docker/) [images](https://hub.docker.com/r/civisanalytics/datascience-python/) for doing data science in Python.
Some people [advocate for the usage of alpine-based images](https://nickjanetakis.com/blog/alpine-based-docker-images-make-a-difference-in-real-world-apps), others [criticize alpine's lack of support for the types of workloads commonly used for data analysis in Python](https://pythonspeed.com/articles/alpine-docker-python/).
Many people bring up [Docker's support for multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/) as a way to keep Docker images small, but in what contexts are multi-stage builds effective?
Is there a simple, generic way to use them without getting into a long discussion on how to optimize Dockerfiles?
This post will discuss all these issues - and more!

<!-- TEASER_END -->

### The values: simplicity and ubiquity

It would be nice to have a simple solution that "just works" in a wide variety of contexts, and that re-uses base images as efficiently as possible across different kinds of projects.
We might be able to engineer a slightly more efficient image with some extra complexity and customization, but to some extent this represents extra work we might end up having to do time and time again as we move from one project to another.
If we use one custom base image across many projects, what happens when we start a new project that requires new dependencies?
If we use a different custom base image for each project, how efficiently will layers be re-used across the different images?
Can you guarantee that end users using only one of your products will "buy in" to your base image system and actually re-use those base images?
These are all tricky questions to answer for a data science developer just looking to Dockerize one or two Python apps - it almost sounds like we're destined to spend a lot of time building out a complex heirarchy of base images just to support a handful of use cases.

### The easy solution: `python:slim`

If your Python project's dependencies are all immediately installable as wheels into a `python:slim` base image, then Dockerizing your application is as simple as:

```dockerfile
FROM python:slim

# install deps
COPY requirements.txt /app/requirements.txt
RUN pip install --no-cache-dir -r /app/requirements.txt

# copy and install app code
COPY . /app
RUN pip install --no-deps /app
```

Installing the requirements in a separate layer before the application code is copied into the image ensures that we don't have to reinstall dependencies every time the application code changes.
Using [a `requirements.txt` file](https://pip.pypa.io/en/stable/user_guide/#requirements-files) to document your dependencies is already a [best practice for Python coding](https://docs.python-guide.org/dev/virtualenvs/#other-notes) - it helps make sure your code is reproducible by allowing you and others to re-create the Python environment your application was developed and tested in.
Other than that, there is nothing complex about this, and the base image we're relying on is both ubiquitous and official.
Even with lots of heavy-weight data science dependencies (e.g., scipy) involved, images made like this can easily be on the order of 200 MB or smaller in terms of compressed size.

If your application depends on packages that don't have wheels and require specific build-time dependencies that aren't present in the `python:slim` base image, the approach above isn't going to work for you, however.

### The lazy (but not bad) solution: `python:latest`

The simplest solution to this "hard to build dependencies" problem is to simply change the base image to `python:latest` and be done with it.
This will cost us about 200 MB in terms of the compressed size of the final image, however, so we might be interested in a better solution.

### The complex solution: `python:slim` + apt-get

An alternative approach is to stick with the `python:slim` base image, but add a bunch of `RUN apt-get install` lines to add the extra dependencies we need at build time.
We might be thinking, "surely we don't need _everything_ in `python:latest` - we will just install exactly what we need."
The main downside of this approach is that you are basically agreeing to manage an extra layer of dependencies beyond your direct Python dependencies as specified in `requirements.txt`.
Deducing what packages you'll need to install is basically a trial-and-error process - Python packages don't have a formal, systematic way of specifying their build-time dependencies.
What happens when `requirements.txt` shrinks?
Do you try all subsets of the system packages installed via `apt-get` in your Dockerfile and keep the smallest subset that works?
It would be nice if we didn't have to manage this extra layer of dependencies.

Another issue with this approach is that you're almost guaranteed to need a large system package like `build-essential` to build many common Python packages anyway, so the actual size savings might be smaller than you expect.

At this point, we might realize the appeal of multi-stage builds - why are we working so hard to optimize the size of image layers we don't even need at runtime?

### The general solution: multi-stage build

What if we had a simple approach that would work for any Python application, even those depending on Python packages that won't build in `python:slim`, but still get `python-slim`-sized (low weight) and `python-slim`-based (ubiquitous) final images?
This is where we finally turn to Docker's [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/).
My recommended pattern for this is based on [this blog post](https://dx13.co.uk/articles/2020/01/02/python-packaging-in-2020/) and looks like this:

```dockerfile
# dep builder: builds wheels for all deps
FROM python:latest AS dep-builder

# build wheels for all deps in requirements.txt
COPY requirements.txt /build/requirements.txt
RUN pip wheel --no-deps -w /build/dist -r /build/requirements.txt

# base image: installs wheels for all dependencies
FROM python:slim AS base

# copy all wheels from builder and install
COPY --from=dep-builder [ "/build/dist/*.whl", "/install/" ]
RUN pip install --no-index /install/*.whl \
    && rm -rf /install

# app builder: builds wheel for just our app
FROM python:latest as app-builder

# copy app code and build wheel for our app
# the only file we've used above this line is requirements.txt
# therefore only the lines below will be re-run when app code changes
COPY . /build
RUN pip wheel --no-deps -w /build/dist /build

# final image: start from base and add just our app
FROM base AS final

# copy all wheels from builder and install
COPY --from=app-builder [ "/build/dist/*.whl", "/install/" ]
RUN pip install --no-index /install/*.whl \
    && rm -rf /install
```

This approach gives us the best of both worlds.
We keep our heavyweight build-time dependencies in `dep-builder` and `app-builder` and move only the built wheels from the builders to our actual image.
We move the dep wheels into `base` before building the application code (which gets added to `final`).
This ensures that when our app code changes without a change in the dependencies, `base` doesn't need to be remade - the only thing we need to do is rebuild our app, copy it to `final`, and install it there.

Note that because we use a heavyweight `app-builder` image to build our app (which is based on `python:latest`), our app is free to depend on things like C extensions and other system libraries during build time.
If our app builds just fine on `python:slim`, we could get rid of the `app-builder` stage - we could instead simply copy our app code into `final` and install it directly from source.

### Caveats

One caveat to this approach is that Python packages may technically call out to non-Python executables installed on the system that are not included in the wheel.
This is common for Python "wrapper" libraries that wrap the API of a non-Python commmand line tool, for example.
In these cases, you can't avoid installing the non-Python command line tool (and all of its dependencies) in the final image.

Another caveat to this wheel-based multi-stage build process is that even though we try to delete the wheels in the `RUN pip install` layer, they are still present in the preceding `COPY --from` layer.
This means that the wheels end up consuming twice as much space, since the wheel is copied `/install` and then the wheel's contents are installed into `site-packages`.
This is hard to avoid, because there's no easy way to perform both the `COPY` and `RUN` commands in the same layer.

### Conclusion

In a nutshell, multi-stage builds allow us to take advantage of the fact that the Python runtime itself is relatively lightweight, even though actually building Python packages might require heavyweight system dependencies.
By keeping the build-time and runtime environments in separate images, we ensure that the runtime image is as small as possible, while simultaneously allowing the build-time requirements to be arbitrarily heavy without impacting the weight of our final image.
While we don't normally think of Python as a language that requires a build step (since Python itself is interpreted rather than compiled), adopting this perspective can help keep our Python Docker images small while avoiding the headache of manually managing and minimizing system-level dependencies.
