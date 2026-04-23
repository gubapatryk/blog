+++
title = "How to quickly create gaussian splats on your Macbook" 
date = 2026-04-23

[taxonomies]
categories = ["Gamedev"] 
tags = ["gaussian-splat", "mac"]
+++

Make your 3D models with ease!

<!-- more -->
{{ responsive(src="pie.jpg", width=500, height=370, alt="Butter cookies") }}

<video width="640" height="360" controls>

  <source src="supersplat.mp4" type="video/mp4">

</video>

## Spinny boy

Here should be embeded playcanvas code:

<div id="viewer" style="width:400px;height:300px;"></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/playcanvas/2.11.8/playcanvas.min.js"></script>

<script>
(function () {

  const container = document.getElementById("viewer");

  const canvas = document.createElement("canvas");
  canvas.style.width = "100%";
  canvas.style.height = "100%";
  container.appendChild(canvas);

  const app = new pc.Application(canvas, {
    mouse: new pc.Mouse(canvas),
    touch: new pc.TouchDevice(canvas)
  });

  app.setCanvasFillMode(pc.FILLMODE_NONE);
  app.setCanvasResolution(pc.RESOLUTION_AUTO);
  app.start();

  // --- camera ---
  const camera = new pc.Entity();
  camera.addComponent("camera", {
    clearColor: new pc.Color(0, 0, 0)
  });
  camera.setLocalPosition(0, 0, 6);
  app.root.addChild(camera);

  // --- splat entity ---
  const splat = new pc.Entity();
  app.root.addChild(splat);

  const SOG_URL = "dzik.sog";

  app.assets.loadFromUrl(SOG_URL, "gsplat", (err, asset) => {
    if (err) {
      console.error(err);
      return;
    }

    splat.addComponent("gsplat", {
      asset: asset
    });
  });

  // --- controls ---
  let rotX = 0;
  let rotY = 0;
  let dragging = false;

  app.mouse.on(pc.EVENT_MOUSEDOWN, () => dragging = true);
  app.mouse.on(pc.EVENT_MOUSEUP, () => dragging = false);

  app.mouse.on(pc.EVENT_MOUSEMOVE, (e) => {
    if (!dragging) return;
    rotY -= e.dx * 0.2;
    rotX -= e.dy * 0.2;
  });

  app.touch.on(pc.EVENT_TOUCHSTART, () => dragging = true);
  app.touch.on(pc.EVENT_TOUCHEND, () => dragging = false);

  app.on("update", (dt) => {
    if (!dragging) {
      rotY += dt * 20; // auto rotate
    }

    splat.setEulerAngles(rotX, rotY, 180);
  });

})();
</script>

End of playcanvas component!