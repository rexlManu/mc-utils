<script>
  import { toast } from "@zerodevx/svelte-toast";

  let inputText = "";
  let shouldCloseTags = false;

  $: convertedText = legacyToMiniMessage(inputText);

  function legacyToMiniMessage(legacyText) {
    const colorCodes = {
      0: "black",
      1: "dark_blue",
      2: "dark_green",
      3: "dark_aqua",
      4: "dark_red",
      5: "dark_purple",
      6: "gold",
      7: "gray",
      8: "dark_gray",
      9: "blue",
      a: "green",
      b: "aqua",
      c: "red",
      d: "light_purple",
      e: "yellow",
      f: "white",
      k: "obfuscated",
      l: "bold",
      m: "strikethrough",
      n: "underline",
      o: "italic",
      r: "reset",
    };

    const regex = /(?:§|&)([0-9a-fk-or])/g;
    const stack = [];

    const closeTags = (skip = 0) => {
      if (!shouldCloseTags) return "";
      let closedTags = "";
      const count = stack.length - skip;
      for (let i = 0; i < count; i++) {
        closedTags += `</${stack.pop()}>`;
      }
      return closedTags;
    };

    const miniMessageText = legacyText.replace(regex, (match, code) => {
      const miniMessageCode = colorCodes[code.toLowerCase()];

      if (miniMessageCode === "reset") {
        return closeTags() + `<${miniMessageCode}>`;
      }

      if (stack.includes(miniMessageCode)) {
        return "";
      }

      if (/[0-9a-f]/.test(code)) {
        const closedTags = closeTags(
          stack.findIndex((tag) => /[0-9a-f]/.test(tag.charAt(0))) + 1
        );
        stack.push(miniMessageCode);
        return closedTags + `<${miniMessageCode}>`;
      }

      stack.push(miniMessageCode);
      return `<${miniMessageCode}>`;
    });

    return miniMessageText + closeTags();
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

<main
  class="w-[60%] grid grid-cols-1 md:grid-cols-2 gap-10 text-center text-white"
>
  <!-- button to toggle shouldCloseTags -->
  <div class="flex flex-col gap-3 col-span-2">
    <h3 class="font-medium text-white text-[20px]">Options</h3>
    <label class="flex items-center gap-2">
      <input
        type="checkbox"
        bind:checked={shouldCloseTags}
        class="h-5 w-5 text-green-500 rounded-md"
      />
      <span class="text-lg">Close tags</span>
    </label>
  </div>

  <div class="flex flex-col gap-3">
    <h3 class="font-medium text-white text-[20px]">Input</h3>
    <textarea
      class="w-full text-lg text-gray-400 font-mono rounded-md p-2 bg-[#141517] resize-none h-auto min-h-[400px]"
      bind:value={inputText}
      on:input
    />
  </div>
  <div class="flex flex-col gap-3">
    <h3 class="font-medium text-white text-[20px]">MiniMessage</h3>
    <div class="relative">
      <textarea
        class="w-full text-lg text-gray-400 rounded-md p-2 bg-[#141517] resize-none h-auto min-h-[400px]"
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
