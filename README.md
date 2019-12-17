This is a static site built with [Nikola](https://getnikola.com/). The `src`
branch contains the source, while the `master` branch contains the built HTML,
which is then hosted by [GitHub Pages](https://pages.github.com/).

To build the site:

    nikola build

To serve it in dev mode:

    nikola serve -b

To autobuild the site:

    nikola auto -b

To deploy to GitHub Pages:

    nikola github_deploy
