# vue-comps-tooltip

a advanced tooltip.

### [Demo](https://vue-comps.github.io/vue-comps-tooltip)

# Install

```sh
npm install --save-dev vue-comps-tooltip
```
or include `build/bundle.js`.

## Usage
```coffee
# in your component
components:
  "tooltip": require("vue-comps-tooltip")
# or, when using bundle.js
components:
  "tooltip": window.vueComps.tooltip
```
```html
<button> Hover
  <tooltip>Content</tooltip>
</button>
```
see [`dev/`](dev/) for examples.

#### Props
| Name | type | default | description |
| ---:| --- | ---| --- |
| offset | Number | 0 | offset to the parent |
| anchor | String | "snwe" | direction of opening, viewport dependet. "s" forces to open down. "sn" would try to open down, the up. |
| class | String | tooltip | class of the `div`|
| ignore-parent | Boolean | false | will not set-up `mouseenter`/`mouseleave` listener on parent |
| is-opened	| Boolean	| false | (two-way) set to open / close|
| transition-in | function | no animation | set animation. Argument: {el,pos,style,cb} |
| transition-out | function | no animation | set animation. Argument: {el,style,cb} |

| parent | element | parentElement | where the tooltip should attach |

#### Events
| Name |  description |
| ---:| --- |
| close |  when received, will close |
| before-open | will be called before open animation |
| opened |  will be called when opened |
| before-close |  will be called before close animation |
| closed |  will be called when closed |



# Development
Clone repository.
```sh
npm install
npm run dev
```
Browse to `http://localhost:8080/`.

## License
Copyright (c) 2016 Paul Pflugradt
Licensed under the MIT license.
