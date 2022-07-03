## How to display text over Particles.JS element?

### Background:

I was testing out [ParticlesJS](https://vincentgarreau.com/particles.js/) on a side-project and I got stuck trying to show text over particles.js

### Problem

Take a look at this HTML code. The problem is, whenever I add something inside `#particles-js` (which is where ParticlesJS's `canvas` gets injected), my animations start "bleeding into" other sections of the page.

This bleeding is clearly visible in case of vertical screen sizes like mobiles.

```
// HTML
<!-- content boundary -->
<section id="particlesjs-container">
<!-- init particlesjs in here -->  
<div id="particles-js">
    <!-- the text I want to show -->
    <h1 id="particlejs-heading" class="text-center"></h1>  
  </div>
</section>

```

I thought of 2 ways to fix this.


## Plan - I (LAZY WAY)

My first plan was to use `@media` CSS queries to manipulate the parent `div` for various screen sizes.

```
@media (min-width: 250px) and (max-width: 300px) {
  #particlejs-heading {
    top: 5vh;
    left: 10vw;
  }
}

@media (min-width: 300px) and (max-width: 400px) {
  #particlejs-heading {
    top: 7vh;
    left: 12vw;
  }
}

@media (min-width: 400px) and (max-width: 400px) {
  #particlejs-heading {
    top: 8vh;
    left: 16vw;
  }
}
// and so on

```

This worked and the text overlay was responding correctly (almost) in various screen sizes. 

HOWEVER, all that changed when I had to change the text. All my alignments were off and they will have to be re-done.  

## Plan - II  (SMART WAY)

[FlexBox](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox) has been around for what feels like forever now and has been integrated into all modern CSS frameworks we use.

> A framework is just a tool that gets the job done. Stop the framework wars!

Here's how I got it working with FlexBox:

```
// HTML
<section id="particlesjs-container">
  <!-- added `row` CSS class which uses FlexBox in Bootstrap -->
  <div id="particles-js" class="row container-fluid">
    <h1 id="particlejs-heading" class="col-12 text-center"></h1>
  </div>
</section>

```

```
// CSS
#particlesjs-container {
  width: 100%;
  height: 78vh;
  background: blue;
}

#particlejs-heading {
  margin-top: 30vh;
  font-size: 5em;
  font-weight: bold;
}

/* secret sauce!! */
canvas {
  position: absolute;
  max-height: 80%;
}

```

And it worked!!

The problem was resolved when I set `position: absolute` to `canvas` so that it is "stuck" within `particlesjs-container` while the text will now auto-adjust thanks to FlexBox. 
