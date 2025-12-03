# This is My Personal Blogsite

This site is built using Hugo, a fast and flexible static site generator. Here, I share whatever I can think of.

## Installation
To run this blog locally, install Hugo and clone the repository:
```sh
# linux
latest_version=$(curl -s https://api.github.com/repos/gohugoio/hugo/releases/latest | grep -oP '"tag_name": "\K(.*)(?=")')
wget https://github.com/gohugoio/hugo/releases/download/${latest_version}/hugo_extended_${latest_version#v}_linux-amd64.deb
sudo dpkg -i hugo_extended_${latest_version#v}_linux-amd64.deb

git clone https://github.com/asdiAdi/blogsite.git
cd blogsite
git submodule update --init --recursive
hugo server
```

Open your browser and visit http://localhost:1313

