---
layout: slides
title: Data Synchronization
permalink: /slides/synching/
---

<section data-markdown data-separator="^\n---\n$" data-separator-vertical="^\n--\n$">
<script type="text/template">

![Bonsai](../../assets/images/bonsai-lettering.svg)

### Data Synchronization
[neurogears.org/st-andrews-2024](https://neurogears.org/st-andrews-2024)
<table style="width: 100%;">
  <tr>
    <th style="vertical-align: middle; width: 50%; height: 100px; padding-left: 100px">
      <img alt="NeuroGEARS" src="../../assets/images/neurogears.svg"/>
    </th>
    <th style="vertical-align: middle; width: 40%; height: 170px; align: right">
      <img alt="St-Andrews" src="../../assets/images/st-andrews.png"/>
    </th>
  </tr>
</table>

---

#### The Ideal

![Single DAQ](../../assets/images/nidaq.jpg)

Single synchronized DAQ at a fixed sampling frequency

---

#### The Reality

<img src="../../assets/images/devices.jpg" height="450" alt="Devices compatible with Bonsai">

Mesh of multi-purpose asynchronous devices

---

#### Example: Measuring reaction times

![Reaction Time Circuit](../../assets/images/reaction-time-circuit.png)

--

###### Zip

![Zip](../../assets/images/zip.svg)

--

###### Delay

![Delay](../../assets/images/delay.svg)

--

###### Repeat

![Delay](../../assets/images/repeat.svg)

--

#### Example: Measuring reaction times

![Reaction Time Workflow](../../assets/images/synching-reaction-time.svg)

---

###### CombineLatest

![CombineLatest](../../assets/images/combinelatest.svg)

--

###### WithLatestFrom

![WithLatestFrom](../../assets/images/withlatestfrom.svg)

--

#### Example: Synchronizing video from two webcams

![Example workflow](../../assets/images/synching-camera.svg)

---

<!-- .element: data-transition="default none" -->
###### Example: Trigger audio on cue

![SelectMany](../../assets/images/selectmany-playsound-1.svg)

--


<!-- .element: data-transition="default none" -->
###### Transform

![Transform](../../assets/images/transform.svg)

--

<!-- .element: data-transition="default none" -->
###### Select

![Select](../../assets/images/select.svg)

--

<!-- .element: data-transition="none default" -->
###### SelectMany

![SelectMany](../../assets/images/selectmany.svg)

--

<!-- .element: data-transition="none default" -->
###### Example: Trigger audio on cue

![SelectMany](../../assets/images/selectmany-playsound-1.svg)

--

<!-- .element: data-transition="none default" -->
###### Example: Trigger audio on cue

![SelectMany](../../assets/images/selectmany-playsound-2.svg)

---

<!-- .element: data-transition="default none" -->
###### Sample

![Sample](../../assets/images/sample.svg)

--

<!-- .element: data-transition="default none" -->
###### WindowTrigger

![WindowTrigger](../../assets/images/windowtrigger.svg)

--

<!-- .element: data-transition="none default" -->
###### Example: Record triggered video

![WindowTrigger](../../assets/images/windowtrigger-recordclip.svg)

</script>
</section>

<section data-markdown data-separator="^\n---\n$" data-separator-vertical="^\n--\n$">
<script type="text/template">

![Bonsai](../../assets/images/bonsai-lettering.svg)

### Questions?
[neurogears.org/st-andrews-2024](https://neurogears.org/st-andrews-2024)
<table style="width: 100%;">
  <tr>
    <th style="vertical-align: middle; width: 50%; height: 100px; padding-left: 100px">
      <img alt="NeuroGEARS" src="../../assets/images/neurogears.svg"/>
    </th>
    <th style="vertical-align: middle; width: 40%; height: 170px; align: right">
      <img alt="St-Andrews" src="../../assets/images/st-andrews.png"/>
    </th>
  </tr>
</table>

</script>
</section>