---
layout: post
title: Intro to ThreeJS... using ThreeJS
tags: webgl javascript
---

I'm a big believer in learning by doing. So when I set out to learn ThreeJS, I did it by making [this sweet slide deck](http://www.dimroc.com/reveal.js-threejs/).
Click through to check out details.

<!--more-->

<canvas id="spinningCube"></canvas>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r71/three.min.js"></script>

<script>
var dimroc = (function() {
  return { gfx: {
    width: 320,
    height: 250
  }};
})();

var renderSpinningCube = function(canvas) {
  var scene = new THREE.Scene();

  var camera = new THREE.PerspectiveCamera( 30, dimroc.gfx.width / dimroc.gfx.height, 1, 1000 );
  camera.position.set(0, 3, 7);
  camera.lookAt( new THREE.Vector3(0,0,0));

  var scale = 2.5;
  var geometry = new THREE.BoxGeometry( scale, scale, scale );
  var material = new THREE.MeshBasicMaterial( { color: 0x000000, wireframe: true, wireframeLinewidth: 3 } );

  var mesh = new THREE.Mesh( geometry, material );
  scene.add( mesh );

  var axisHelper = new THREE.AxisHelper(50);
  scene.add( axisHelper );

  var renderer = new THREE.WebGLRenderer({canvas: canvas, antialias: true, alpha: true});
  renderer.setSize( dimroc.gfx.width, dimroc.gfx.height );

  function animate() {
    requestAnimationFrame( animate, canvas );
    mesh.rotation.y += 0.008;
    renderer.render( scene, camera );
  }

  animate();
}

renderSpinningCube($('#spinningCube')[0]);

</script>

Using [reveal.js](http://lab.hakim.se/reveal-js/#/), an html framework for presentations, I added self-contained javascript ThreeJS samples on each slide to
demonstrate a particular feature of ThreeJS.

Here is the standard spinning cube example:

{% highlight javascript %}

(function() {

  window.samples.spinning_cube = {

    initialize: function(canvas) {
      var scene = new THREE.Scene();

      var camera = new THREE.PerspectiveCamera( 30, sample_defaults.width / sample_defaults.height, 1, 1000 );
      camera.position.set(0, 3, 7);
      camera.lookAt( new THREE.Vector3(0,0,0));

      var scale = 2.5;
      var geometry = new THREE.CubeGeometry( scale, scale, scale );
      var material = new THREE.MeshBasicMaterial( { color: 0xdddddd } );

      var mesh = new THREE.Mesh( geometry, material );
      scene.add( mesh );

      var renderer = new THREE.WebGLRenderer({canvas: canvas, antialias: true});
      renderer.setSize( sample_defaults.width, sample_defaults.height );

      var instance = { active: false };
      function animate() {
        requestAnimationFrame( animate, canvas );
        if(!instance.active || sample_defaults.paused) return;

        mesh.rotation.y += 0.008;

        renderer.render( scene, camera );
      }

      animate();
      return instance;
    }
  };
})();

{% endhighlight %}

By tagging slides with a `data-sample` attribute, the corresponding javascript example kicks in and does its WebGL magic.

{% highlight html %}
<section>
  <canvas data-sample="spinning_cube" style="display:inline"></canvas>
</section>
{% endhighlight %}

See all the ThreeJS javascript samples [here](https://github.com/dimroc/reveal.js-threejs/tree/gh-pages/js/samples).

Check out the actual [Intro to ThreeJS Slideshow](http://www.dimroc.com/reveal.js-threejs/#/). [Source Code](https://github.com/dimroc/reveal.js-threejs).
