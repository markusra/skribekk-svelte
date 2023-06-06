<script lang="ts">
  import { getContext, onMount } from "svelte";
  import toast, { Toaster } from "svelte-french-toast";

  import WordSelectPopup from "./popups/WordSelectPopup.svelte";
  import MessagePopup from "./popups/MessagePopup.svelte";
  const { open } = getContext("simple-modal");

  enum Color {
    BLACK = "black",
    BLUE = "blue",
    GREEN = "green",
    RED = "red",
  }

  type Pos = { x: number; y: number };

  type DrawObject = {
    pos: Pos;
    color: Color;
  };

  type MessageType =
    | "setName"
    | "startGame"
    | "setWord"
    | "gameStarted"
    | "draw"
    | "guess"
    | "wordGuessed";

  let socket = new WebSocket("ws://192.168.0.110:3333");

  let canvasElement: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D;

  let inputField: HTMLInputElement;

  let prevPos: { x?: number; y?: number } = { x: undefined, y: undefined };
  let strokeColor: Color = Color.BLACK;

  let gameStarted = false;
  let wordGuess = "";

  const sendMessage = (type: MessageType, data: any) =>
    socket.send(JSON.stringify({ type, data }));

  const onWordSelect = (word: string) => {
    sendMessage("setWord", word);
    clearCanvas();
    inputField.value = word;
  };

  onMount(() => {
    ctx = canvasElement.getContext("2d") as CanvasRenderingContext2D;

    let draw = false;

    // Need this since we set width and height to 100 % with CSS
    canvasElement.width = canvasElement.offsetWidth;
    canvasElement.height = canvasElement.offsetHeight;

    // Smooth lines
    ctx.lineCap = "round";

    ctx.lineWidth = 5;

    window.addEventListener("mousedown", (e) => {
      draw = true;

      const offset = canvasElement && canvasElement.getBoundingClientRect();
      prevPos = { x: e.x - offset.left, y: e.y - offset.top };
    });

    window.addEventListener("mouseup", () => {
      draw = false;
    });

    canvasElement.addEventListener("mousemove", (e) => {
      const offset = canvasElement && canvasElement.getBoundingClientRect();

      if (draw) {
        sendMessage("draw", {
          pos: { x: e.x - offset.left, y: e.y - offset.top },
          prevPos,
          color: strokeColor,
        });
      }
    });
  });

  function clearCanvas() {
    ctx.clearRect(0, 0, canvasElement.width, canvasElement.height);
  }

  function setColor(color: Color) {
    strokeColor = color;
  }

  function startGame() {
    sendMessage("startGame", null);
  }

  socket.onopen = () => {
    console.log("[open] Connection established");
    sendMessage("setName", "Markus");
  };

  socket.onmessage = (event) => {
    event.data.text().then((msg) => {
      const { type, data }: { type: MessageType; data: any } = JSON.parse(msg);

      switch (type) {
        case "setName":
          toast(`${data} joined the game!`);
          break;
        case "startGame":
          open(
            WordSelectPopup,
            { words: data, onWordSelect },
            { closeButton: false, closeOnEsc: false, closeOnOuterClick: false }
          );
          break;
        case "gameStarted":
          open(MessagePopup, {
            message: "<span>🤓</span> Game started! Guess the word.",
          });
          clearCanvas();
          gameStarted = true;
          break;
        case "draw":
          const { pos, prevPos: serverPrevPos, color }: DrawObject = data;
          ctx.strokeStyle = color;

          ctx.beginPath();
          ctx.moveTo(serverPrevPos.x, serverPrevPos.y);
          ctx.lineTo(pos.x, pos.y);
          ctx.stroke();

          prevPos = { x: pos.x, y: pos.y };
          break;
        case "wordGuessed":
          open(MessagePopup, {
            message: `<span>🎉</span> ${data.name} guessed '${data.word}' correctly!`,
          });
          gameStarted = false;
          break;
      }
    });
  };

  socket.onclose = (event) => {
    if (event.wasClean) {
      console.log(
        `[close] Connection closed cleanly, code=${event.code} reason=${event.reason}`
      );
    } else {
      // e.g. server process killed or network down
      // event.code is usually 1006 in this case
      console.log("[close] Connection died");
    }
  };

  socket.onerror = (error) => {
    console.log(`[error] Connection error`);
  };
</script>

<div class="header">
  <h1>Skribekk</h1>
  <button class="action" type="button" on:click={startGame}>Start Game!</button>
</div>
<div class="toolbar">
  <button class="color black" on:click={() => setColor(Color.BLACK)} />
  <button class="color blue" on:click={() => setColor(Color.BLUE)} />
  <button class="color green" on:click={() => setColor(Color.GREEN)} />
  <button class="color red" on:click={() => setColor(Color.RED)} />
  <button
    type="button"
    on:click={() => {
      !gameStarted && clearCanvas();
    }}>Clear</button
  >
</div>
<canvas bind:this={canvasElement} />
<input
  class="guess-input"
  type="text"
  placeholder="Guess!"
  bind:this={inputField}
  bind:value={wordGuess}
  on:keydown={(e) => {
    if (e.key === "Enter") {
      sendMessage("guess", wordGuess);
      inputField.value = "";
    }
  }}
/>

<Toaster />
