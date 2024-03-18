---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides, markdown enabled
title: WebRTC-QoE
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply any unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
---
# Estimating WebRTC Video QoE Metrics Without Using Application Headers

<br>

## Taveesh Sharma, Tarun Mangla et.al *IMC 2023*
<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
transition: fade-out
layout: two-cols
layoutClass: gap-16
---

# What is WebRTC?

WebRTC, short for Web Real-Time Communication, is both an API and a Protocol.
- **Signaling** 
  - SDP (Session Description Protocol)
- **Connecting** 
  - ICE (Interactive Connectivity Establishment)
- **Securing**
  - DTLS (Datagram Transport Layer Security) 
  - SRTP (Secure Real-Time Transport Protocol)
- **Communicating**
  - <span v-mark.underline.red="2">RTP (Real-time Transport Protocol)</span>
  - SCTP(Stream Control Transmission Protocol)

::right::
<div v-click>
```mermaid {theme: 'neutral', scale: 0.9}
graph LR
A[WebRTC Agent]
A --> B[SCTP Agent]
A --> C[DTLS Agent]
A --> D[ICE Agent]
D --> E[TURN Agent]
D --> F[STUN Protocol]
A --> G[SRTP Agent]
G --> H[RCTP]
G --> I[RTP]
A --> K[SDP]
```
</div>

<div class="abs-br flex gap-2">
  <a href="https://github.com/webrtc-for-the-curious/webrtc-for-the-curious" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

<!--
Here is another comment.
-->

---
transition: slide-up
level: 2
---

# WebRTC based Video Conference Applications (VCAs)

<div grid="~ cols-2 gap-2" m="t-2">
<img src="/zoom.png" style="max-width: 90%; max-height: 100%;">
<img src="/meet.jpg" style="max-width: 100%; max-height: 100%;">
<img src="/webex.jpg" style="max-width: 90%; max-height: 100%;">
<img src="/Teams.png" style="max-width: 100%; max-height: 100%;">
</div>


<!--
Here is another comment.
-->

---
transition: slide-left
---

# Motivation
- Quality of Experience (QoE) is critical for VCAs' using
- QoE can be improved by optimizing both the end hosts and <span v-mark.circle.orange="1">the network</span>

<div v-click>
</div>

<div v-click>

How can network operators know about QoE metrices?

  - Existing VCA QoE estimation methods use passive measurements of application-level RTP headers
    - They have no access to end hosts
    - Some apps may use custom RTP protocols → need special methods
    - VPN → encrypted application headers

</div>

<div v-click>

**GOAL**: Can we use <span v-mark.underline.red="3">IP/UDP headers</span> to infer QoE?

</div>

<div v-click>

```mermaid {theme: 'neutral', scale: 0.8}
graph TD
A[IP/UDP Heuristic]
B[IP/UDP ML]
C[RTP Heuristic]
D[RTP ML]
```

</div>

---
transition: slide-left
layout: two-cols
layoutClass: gap-16
---

# QoE metrices
Focused on video QoE for VCAs
> previous work has already estimated audio QoE for VoIP

<div v-click>

- Frame Rate/FPS (Smoothness)

</div>

<div v-click>

- Bitrate (Data Transfer Rate)

</div>

<div v-click>

- Frame Jitter (Consistency)

</div>

<div v-click>

- Resolution (Detail)

</div>

::right::
<br>
<br>
<br>

<div v-click=1>

<img src="/fps.gif">

</div>


---
transition: fade-out
---
# Method-Step 1
The first step involves isolating the video traffic from the audio component

<div v-click>

Three kinds of *payload type*:
- PT = 111: audio encoded using OPUS → \[89, 385\] bytes
- PT = 102: video encoded using H.264 → 99% > 564 bytes
- PT = 103: video retransmissions → 92% 304 bytes: keep-alive messages

</div>

<div v-click>
<img src="/v-a.png" style="margin: auto; max-width: 50%; max-height: 100%;">
</div>

<div v-click>

> Use a size threshold $V_{min}$ to identify video packets

</div>


---
transition: slide-up
---
# Method-Step 2
Identifying the video frame boundaries (by identifying frame end time) without using RTP headers

<div v-click>

- **Key Insight 1**: inter-departure time within the frame < inter-departure time across the frame

</div>

<img v-click src="/depature-time.png" style="margin: auto;">

<div v-click>

- Problems:
  - packet timings can change when packets traverse along the network
  - patterns in inter-departure time may not appear in inter-arrival time
  - happens when congestion or packet loss occurs

</div>

<img v-click src="/actual.png" style="margin: auto;">

---
transition: slide-up
---
# Method-Step 2
Identifying the video frame boundaries (by identifying frame end time) without using RTP headers

<div v-click>

- **Key Insight 2**: there are unique patterns in packet sizes
  - packet sizes tend to resemble those within the same frame and differ from those in consecutive frames
  - VCAs typically fragment a frame into equal-sized packets
    - Forward Error Correction (FEC) mechanisms
      - bandwidth-efficient when packets in a frame have equal length

</div>

<img v-click src="/packet-size.png" style="margin: auto; max-width: 50%;">

---
transition: slide-up
---
# Method-Step 2
Identifying the video frame boundaries (by identifying frame end time) without using RTP headers

<div grid="~ cols-2 gap-1" m="t-2">

<div v-click>

- **Frame boundary estimation**: 
  - packet size threshold $\Delta^{max}_{size}$
  - not sufficient to compare only consecutive packets as packets can arrive out of order
    - compare with up to $N^{max}$ packets taht arrived before this packet
    - $N^{max}$ should be set carefully


</div>

<img v-click src="/algo1.png">

</div>

---
transition: fade-out
---
# Heuristic method

<img src="/ppt-show.png">

---
transition: fade-out
---
# IP/UDP Heuristic Failure Cases
Wrong estimation of frame boundaries

## Three cases

<br>

<img v-click src="/failure.png">

---
transition: slide-up
---

Hover on the bottom-left corner to see the navigation's controls panel, [learn more](https://sli.dev/guide/navigation.html)

## Keyboard Shortcuts

|                                                     |                             |
| --------------------------------------------------- | --------------------------- |
| <kbd>right</kbd> / <kbd>space</kbd>                 | next animation or slide     |
| <kbd>left</kbd>  / <kbd>shift</kbd><kbd>space</kbd> | previous animation or slide |
| <kbd>up</kbd>                                       | previous slide              |
| <kbd>down</kbd>                                     | next slide                  |

<!-- https://sli.dev/guide/animations.html#click-animations -->
<img
  v-click
  class="absolute -bottom-9 -left-7 w-80 opacity-50"
  src="https://sli.dev/assets/arrow-bottom-left.svg"
  alt=""
/>
<p v-after class="absolute bottom-23 left-45 opacity-30 transform -rotate-10">Here!</p>

---
layout: two-cols
layoutClass: gap-16
---

# Table of contents

You can use the `Toc` component to generate a table of contents for your slides:

```html
<Toc minDepth="1" maxDepth="1"></Toc>
```

The title will be inferred from your slide content, or you can override it with `title` and `level` in your frontmatter.

::right::

<Toc v-click minDepth="1" maxDepth="2"></Toc>

---
layout: image-right
image: https://cover.sli.dev
---

# Code

Use code snippets and get the highlighting directly, and even types hover![^1]

```ts {all|5|7|7-8|10|all} twoslash
// TwoSlash enables TypeScript hover information
// and errors in markdown code blocks
// More at https://shiki.style/packages/twoslash

import { computed, ref } from 'vue'

const count = ref(0)
const doubled = computed(() => count.value * 2)

doubled.value = 2
```

<arrow v-click="[4, 5]" x1="350" y1="310" x2="195" y2="334" color="#953" width="2" arrowSize="1" />

<!-- This allow you to embed external code blocks -->
<<< @/snippets/external.ts#snippet

<!-- Footer -->
[^1]: [Learn More](https://sli.dev/guide/syntax.html#line-highlighting)

<!-- Inline style -->
<style>
.footnotes-sep {
  @apply mt-5 opacity-10;
}
.footnotes {
  @apply text-sm opacity-75;
}
.footnote-backref {
  display: none;
}
</style>

<!--
Notes can also sync with clicks

[click] This will be highlighted after the first click

[click] Highlighted with `count = ref(0)`

[click:3] Last click (skip two clicks)
-->

---
level: 2
---

# Shiki Magic Move

Powered by [shiki-magic-move](https://shiki-magic-move.netlify.app/), Slidev supports animations across multiple code snippets.

Add multiple code blocks and wrap them with <code>````md magic-move</code> (four backticks) to enable the magic move. For example:

````md magic-move
```ts {*|2|*}
// step 1
const author = reactive({
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery'
  ]
})
```

```ts {*|1-2|3-4|3-4,8}
// step 2
export default {
  data() {
    return {
      author: {
        name: 'John Doe',
        books: [
          'Vue 2 - Advanced Guide',
          'Vue 3 - Basic Guide',
          'Vue 4 - The Mystery'
        ]
      }
    }
  }
}
```

```ts
// step 3
export default {
  data: () => ({
    author: {
      name: 'John Doe',
      books: [
        'Vue 2 - Advanced Guide',
        'Vue 3 - Basic Guide',
        'Vue 4 - The Mystery'
      ]
    }
  })
}
```

Non-code blocks are ignored.

```vue
<!-- step 4 -->
<script setup>
const author = {
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery'
  ]
}
</script>
```
````

---

# Components

<div grid="~ cols-2 gap-4">
<div>

You can use Vue components directly inside your slides.

We have provided a few built-in components like `<Tweet/>` and `<Youtube/>` that you can use directly. And adding your custom components is also super easy.

```html
<Counter :count="10" />
```

<!-- ./components/Counter.vue -->
<Counter :count="10" m="t-4" />

Check out [the guides](https://sli.dev/builtin/components.html) for more.

</div>
<div>

```html
<Tweet id="1390115482657726468" />
```

<Tweet id="1390115482657726468" scale="0.65" />

</div>
</div>

<!--
Presenter note with **bold**, *italic*, and ~~striked~~ text.

Also, HTML elements are valid:
<div class="flex w-full">
  <span style="flex-grow: 1;">Left content</span>
  <span>Right content</span>
</div>
-->

---
class: px-20
---

# Themes

Slidev comes with powerful theming support. Themes can provide styles, layouts, components, or even configurations for tools. Switching between themes by just **one edit** in your frontmatter:

<div grid="~ cols-2 gap-2" m="t-2">

```yaml
---
theme: default
---
```

```yaml
---
theme: seriph
---
```

<img border="rounded" src="https://github.com/slidevjs/themes/blob/main/screenshots/theme-default/01.png?raw=true" alt="">

<img border="rounded" src="https://github.com/slidevjs/themes/blob/main/screenshots/theme-seriph/01.png?raw=true" alt="">

</div>

Read more about [How to use a theme](https://sli.dev/themes/use.html) and
check out the [Awesome Themes Gallery](https://sli.dev/themes/gallery.html).

---

# Clicks Animations

You can add `v-click` to elements to add a click animation.

<div v-click>

This shows up when you click the slide:

```html
<div v-click>This shows up when you click the slide.</div>
```

</div>

<br>

<v-click>

The <span v-mark.red="3"><code>v-mark</code> directive</span>
also allows you to add
<span v-mark.circle.orange="4">inline marks</span>
, powered by [Rough Notation](https://roughnotation.com/):

```html
<span v-mark.underline.orange>inline markers</span>
```

</v-click>

<div mt-20 v-click>

[Learn More](https://sli.dev/guide/animations#click-animations)

</div>

---
preload: false
---

# Motions

Motion animations are powered by [@vueuse/motion](https://motion.vueuse.org/), triggered by `v-motion` directive.

```html
<div
  v-motion
  :initial="{ x: -80 }"
  :enter="{ x: 0 }">
  Slidev
</div>
```

<div class="w-60 relative mt-6">
  <div class="relative w-40 h-40">
    <img
      v-motion
      :initial="{ x: 800, y: -100, scale: 1.5, rotate: -50 }"
      :enter="final"
      class="absolute top-0 left-0 right-0 bottom-0"
      src="https://sli.dev/logo-square.png"
      alt=""
    />
    <img
      v-motion
      :initial="{ y: 500, x: -100, scale: 2 }"
      :enter="final"
      class="absolute top-0 left-0 right-0 bottom-0"
      src="https://sli.dev/logo-circle.png"
      alt=""
    />
    <img
      v-motion
      :initial="{ x: 600, y: 400, scale: 2, rotate: 100 }"
      :enter="final"
      class="absolute top-0 left-0 right-0 bottom-0"
      src="https://sli.dev/logo-triangle.png"
      alt=""
    />
  </div>

  <div
    class="text-5xl absolute top-14 left-40 text-[#2B90B6] -z-1"
    v-motion
    :initial="{ x: -80, opacity: 0}"
    :enter="{ x: 0, opacity: 1, transition: { delay: 2000, duration: 1000 } }">
    Slidev
  </div>
</div>

<!-- vue script setup scripts can be directly used in markdown, and will only affects current page -->
<script setup lang="ts">
const final = {
  x: 0,
  y: 0,
  rotate: 0,
  scale: 1,
  transition: {
    type: 'spring',
    damping: 10,
    stiffness: 20,
    mass: 2
  }
}
</script>

<div
  v-motion
  :initial="{ x:35, y: 40, opacity: 0}"
  :enter="{ y: 0, opacity: 1, transition: { delay: 3500 } }">

[Learn More](https://sli.dev/guide/animations.html#motion)

</div>

---

# LaTeX

LaTeX is supported out-of-box powered by [KaTeX](https://katex.org/).

<br>

Inline $\sqrt{3x-1}+(1+x)^2$

Block
$$ {1|3|all}
\begin{array}{c}

\nabla \times \vec{\mathbf{B}} -\, \frac1c\, \frac{\partial\vec{\mathbf{E}}}{\partial t} &
= \frac{4\pi}{c}\vec{\mathbf{j}}    \nabla \cdot \vec{\mathbf{E}} & = 4 \pi \rho \\

\nabla \times \vec{\mathbf{E}}\, +\, \frac1c\, \frac{\partial\vec{\mathbf{B}}}{\partial t} & = \vec{\mathbf{0}} \\

\nabla \cdot \vec{\mathbf{B}} & = 0

\end{array}
$$

<br>

[Learn more](https://sli.dev/guide/syntax#latex)

---

# Diagrams

You can create diagrams / graphs from textual descriptions, directly in your Markdown.

<div class="grid grid-cols-4 gap-5 pt-4 -mb-6">

```mermaid {scale: 0.5, alt: 'A simple sequence diagram'}
sequenceDiagram
    Alice->John: Hello John, how are you?
    Note over Alice,John: A typical interaction
```

```mermaid {theme: 'neutral', scale: 0.8}
graph TD
B[Text] --> C{Decision}
C -->|One| D[Result 1]
C -->|Two| E[Result 2]
```

```mermaid
mindmap
  root((mindmap))
    Origins
      Long history
      ::icon(fa fa-book)
      Popularisation
        British popular psychology author Tony Buzan
    Research
      On effectivness<br/>and features
      On Automatic creation
        Uses
            Creative techniques
            Strategic planning
            Argument mapping
    Tools
      Pen and paper
      Mermaid
```

```plantuml {scale: 0.7}
@startuml

package "Some Group" {
  HTTP - [First Component]
  [Another Component]
}

node "Other Groups" {
  FTP - [Second Component]
  [First Component] --> FTP
}

cloud {
  [Example 1]
}

database "MySql" {
  folder "This is my folder" {
    [Folder 3]
  }
  frame "Foo" {
    [Frame 4]
  }
}

[Another Component] --> [Example 1]
[Example 1] --> [Folder 3]
[Folder 3] --> [Frame 4]

@enduml
```

</div>

[Learn More](https://sli.dev/guide/syntax.html#diagrams)

---
src: ./pages/multiple-entries.md
hide: false
---

---

# Monaco Editor

Slidev provides built-in Moanco Editor support.

Add `{monaco}` to the code block to turn it into an editor:

```ts {monaco}
import { ref } from 'vue'
import hello from './external'

const code = ref('const a = 1')
hello()
```

Use `{monaco-run}` to create an editor that can execute the code directly in the slide:

```ts {monaco-run}
function fibonacci(n: number): number {
  return n <= 1
    ? n
    : fibonacci(n - 1) + fibonacci(n - 2) // you know, this is NOT the best way to do it :P
}

console.log(Array.from({ length: 10 }, (_, i) => fibonacci(i + 1)))
```

---
layout: center
class: text-center
---

# Learn More

[Documentations](https://sli.dev) · [GitHub](https://github.com/slidevjs/slidev) · [Showcases](https://sli.dev/showcases.html)
