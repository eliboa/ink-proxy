# Ink Proxy
A simple tool to extract iksm_session token from Nintendo Switch Online app
![Gif](http://splatoon.eu/ink-proxy/screen.gif)

## Download
Latest releases are available [here](https://github.com/eliboa/ink-proxy/releases) 

## How does it work ?
This is an electron app that uses [mitm proxy](https://github.com/mitmproxy/mitmproxy) to create a proxy on your computer, then sniff https packets from Splatnet on your phone (NSO app) and retrieve your iksm session.

## Use / Related
iksm_session token can be used to access the Splatnet API (Splatoon 2).

* [NintendoSwitchRESTAPI](https://github.com/ZekeSnider/NintendoSwitchRESTAPI) Reverse engineered REST API used in the Nintendo Switch app for iOS. Includes documentation on Splatoon 2's API.
* [stat.ink](https://github.com/fetus-hina/stat.ink) 
* [splatnet2statink](https://github.com/frozenpandaman/splatnet2statink)
