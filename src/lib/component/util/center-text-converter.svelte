<script>
  import { toast } from "@zerodevx/svelte-toast";

  let characters = {
    A: 5,
    a: 5,
    B: 5,
    b: 5,
    C: 5,
    c: 5,
    D: 5,
    d: 5,
    E: 5,
    e: 5,
    F: 5,
    f: 4,
    G: 5,
    g: 5,
    H: 5,
    h: 5,
    I: 3,
    i: 1,
    J: 5,
    j: 5,
    K: 5,
    k: 4,
    L: 5,
    l: 1,
    M: 5,
    m: 5,
    N: 5,
    n: 5,
    O: 5,
    o: 5,
    P: 5,
    p: 5,
    Q: 5,
    q: 5,
    R: 5,
    r: 5,
    S: 5,
    s: 5,
    T: 5,
    t: 4,
    U: 5,
    u: 5,
    V: 5,
    v: 5,
    W: 5,
    w: 5,
    X: 5,
    x: 5,
    Y: 5,
    y: 5,
    Z: 5,
    z: 5,
    1: 5,
    2: 5,
    3: 5,
    4: 5,
    5: 5,
    6: 5,
    7: 5,
    8: 5,
    9: 5,
    0: 5,
    "!": 1,
    "@": 6,
    "#": 5,
    $: 5,
    "%": 5,
    "^": 5,
    "&": 5,
    "*": 5,
    "(": 4,
    ")": 4,
    "-": 5,
    _: 5,
    "+": 5,
    "=": 5,
    "{": 4,
    "}": 4,
    "[": 3,
    "]": 3,
    ":": 1,
    ";": 1,
    '"': 3,
    "'": 1,
    "<": 4,
    ">": 4,
    "?": 5,
    "/": 5,
    "\\": 5,
    "|": 1,
    "~": 5,
    "`": 2,
    ".": 1,
    ",": 1,
    " ": 3,
    default: 4,
  };

  function getLength(char) {
    return characters[char] ?? characters["default"];
  }

  function getBoldLength(char) {
    let buffer = char === " " ? 0 : 1;
    return getLength(char) + buffer;
  }

  function getMessageLength(message) {
    let messagePxSize = 0;
    let previousCode = false;
    let isBold = false;
    let previousTagOpening = false;
    let tagName = "";
    let tagParameters = false;

    for (let i = 0; i < message.length; i++) {
      let char = message[i];
      if (char === "<") {
        previousTagOpening = true;
        continue;
      } else if (previousTagOpening) {
        if (char === ">") {
          previousTagOpening = false;
          if (tagName === "bold" || tagName === "b") {
            isBold = true;
          }
          if (tagName === "/bold" || tagName === "/b") {
            isBold = false;
          }

          if (tagName === "reset") {
            isBold = false;
          }
          continue;
        }
        if (tagParameters) continue;
        if (char === ":") {
          tagParameters = true;
          continue;
        }

        tagName += char;
      } else if (char === "&") {
        previousCode = true;
        continue;
      } else if (previousCode) {
        previousCode = false;
        isBold = char === "l" || char === "L";
        if (isBold) continue;
      } else {
        let length = isBold ? getBoldLength(char) : getLength(char);
        messagePxSize += length + 1;
      }
    }
    return messagePxSize;
  }

  function centerMessage(message, length) {
    let messagePxSize = getMessageLength(message);
    let halvedMessageSize = messagePxSize / 2;
    let toCompensate = length - halvedMessageSize;
    let spaceLength = getLength(" ") + 1;
    let compensated = 0;
    let sb = "";

    while (compensated < toCompensate) {
      sb += " ";
      compensated += spaceLength;
    }
    return sb + message;
  }

  let inputText = "";
  let lineLength = 154;

  $: convertedText = convertToCenteredText(inputText);

  function convertToCenteredText(text) {
    let centered = "";
    let lines = text.split("\n");
    for (let i = 0; i < lines.length; i++) {
      let line = lines[i];
      if (!(i >= lines.length - 1 && (line === "\r" || line === ""))) {
        let nl = centerMessage(line, lineLength);
        centered += nl += "\n";
      }
    }
    return centered;
  }

  function copyValue() {
    navigator.clipboard.writeText(convertedText);
    copyToast();
  }

  function copyToast() {
    toast.push("Copied successfully!", {
      theme: {
        "--toastColor": "mintcream",
        "--toastBackground": "rgba(72,187,120,0.9)",
        "--toastBarBackground": "#2F855A",
      },
    });
  }
</script>

<main class="w-[60%] grid grid-cols-1 gap-10 text-center text-white">
  <div class="flex items-center space-x-4">
    <label class="font-medium text-white">Line Length:</label>
    <input
      class="w-[160px] py-2.5 px-0 text-sm bg-transparent border-0 border-b-2 border-gray-200 text-gray-400 dark:border-gray-700 focus:outline-none focus:ring-0 focus:border-gray-200 peer s-YstiPkdVUP5J"
      type="number"
      bind:value={lineLength}
      on:input
    />

    <button
      class="text-sm px-4 py-1.5 button bg-[#141517]"
      on:click={() => {
        lineLength = 127;
      }}
    >
      MOTD
    </button>
    <button
      class="text-sm px-4 py-1.5 button bg-[#141517]"
      on:click={() => {
        lineLength = 154;
      }}
    >
      Chat
    </button>
  </div>
  <div class="flex flex-col gap-3">
    <h3 class="font-medium text-white text-[20px]">Input</h3>
    <textarea
      class="w-full text-lg text-gray-400 font-mono rounded-md p-2 bg-[#141517] resize-none h-auto min-h-[200px]"
      bind:value={inputText}
      on:input
    />
  </div>
  <div class="flex flex-col gap-3">
    <h3 class="font-medium text-white text-[20px]">Centered Text</h3>
    <div class="relative">
      <textarea
        class="w-full text-lg text-gray-400 rounded-md p-2 bg-[#141517] resize-none h-auto min-h-[200px]"
        disabled
        on:copy={copyToast}>{convertedText}</textarea
      >
      <button
        class="text-sm px-4 py-1.5 absolute right-5 bottom-6 button bg-[#141517]"
        on:click={copyValue}>Copy</button
      >
    </div>
  </div>
</main>
