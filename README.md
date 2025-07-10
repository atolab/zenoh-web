# Eclipse zenoh's Website

The website for the Eclipse zenoh project. Lives at [http://zenoh.io](https://zenoh.io).

## Getting Started

The website should build with the latest version of Hugo, the last tested is [Hugo 0.145.0](http://gohugo.io). 
On MacOS you can install Hugo as described below, in any case refer to [Hugo Documentation](http://gohugo.io) 
for installation instructioins

```sh
brew update && brew install hugo
```

Then, get the website running locally:

```sh
git clone https://github.com/atolab/zenoh-web
cd zenoh-web
hugo server
```

Then visit [http://localhost:1313](http://localhost:1313).

## License

This project is licensed under the [Eclipse Public License 2.0](LICENSE)
or the [Apache License 2.0](LICENSE).

## Credits

This website design is inspired from the [tokio-rs website](https://github.com/tokio-rs/website)
which is licensed under [MIT License](LICENSE-tokio-rs).
