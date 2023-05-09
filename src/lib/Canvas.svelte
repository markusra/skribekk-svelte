<script lang="ts">
  import { onMount } from "svelte";

  enum Color {
    BLACK = "black",
    BLUE = "blue",
    GREEN = "green",
    RED = "red",
  }

  let canvasElement: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D;

  onMount(() => {
    ctx = canvasElement.getContext("2d") as CanvasRenderingContext2D;
    let prevX: number | null = null;
    let prevY: number | null = null;

    let draw = false;

    // Need this since we set width and height to 100 % with CSS
    canvasElement.width = canvasElement.offsetWidth;
    canvasElement.height = canvasElement.offsetHeight;

    // Smooth lines
    ctx.lineCap = "round";

    ctx.lineWidth = 5;

    window.addEventListener("mousedown", () => {
      draw = true;
    });

    window.addEventListener("mouseup", () => {
      draw = false;
    });

    window.addEventListener("mousemove", (e) => {
      const offsetX = canvasElement.getBoundingClientRect().left;
      const offsetY = canvasElement.getBoundingClientRect().top;

      if (prevX === null || prevY === null || !draw) {
        prevX = e.clientX - offsetX;
        prevY = e.clientY - offsetY;
        return;
      }

      let currentX = e.clientX - offsetX;
      let currentY = e.clientY - offsetY;

      ctx.beginPath();
      ctx.moveTo(prevX, prevY);
      ctx.lineTo(currentX, currentY);
      ctx.stroke();

      prevX = currentX;
      prevY = currentY;
    });
  });

  function clearCanvas() {
    ctx.clearRect(0, 0, canvasElement.width, canvasElement.height);
  }

  function setColor(color: Color) {
    ctx.strokeStyle = color;
  }
</script>

<div class="tools">
  <button class="color black" on:click={() => setColor(Color.BLACK)} />
  <button class="color blue" on:click={() => setColor(Color.BLUE)} />
  <button class="color green" on:click={() => setColor(Color.GREEN)} />
  <button class="color red" on:click={() => setColor(Color.RED)} />
  <button type="button" on:click={clearCanvas}>Clear</button>
</div>
<canvas bind:this={canvasElement} />
