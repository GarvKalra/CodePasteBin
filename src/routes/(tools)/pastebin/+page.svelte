<!-- 
   +page.svelte: This file is responsible for displaying the main page of the Pastebin tool, including the form to create a new paste and a list of recent pastes.
   
   +page.server.js: This file contains the server-side logic for handling the creation of new pastes and loading all pastes.
     
   api/+server.js: This file manages API endpoints for creating and fetching pastes.
   
   [id]/+page.svelte: This file handles displaying a specific paste, including decryption if needed.

    [id]/+server.js: This file manages fetching a specific paste by its ID.

	snippets/+server.js: This file handles the creation of new code snippets
  -->
<script lang="ts">
  export let data: { pastes: any[] };
  import Prism from "prismjs";
  import "prismjs/themes/prism-tomorrow.css";
  import "prismjs/components/prism-javascript";
  import "prismjs/components/prism-css";
  import "prismjs/components/prism-markup";


  import { Dropdown, DropdownItem } from "flowbite-svelte";
  import { javascript } from "svelte-highlight/languages";

  let selectedLanguage = "markup";
  function formatExpirationTime(expirationTimestamp: number | null): string {
  if (expirationTimestamp === null) {
    return "Never";
  }

  const now = Date.now();
  const secondsRemaining = Math.floor((expirationTimestamp - now) / 1000);

  if (secondsRemaining <= 0) {
    return "Expired";
  } else if (secondsRemaining < 60) {
    return `${secondsRemaining} seconds`;
  } else if (secondsRemaining < 3600) {
    return `${Math.floor(secondsRemaining / 60)} minutes`;
  } else if (secondsRemaining < 86400) {
    return `${Math.floor(secondsRemaining / 3600)} hours`;
  } else {
    return `${Math.floor(secondsRemaining / 86400)} days`;
  }
}
  function handleFileSelection(event: Event) {
    const input = event.target as HTMLInputElement;
    if (input.files && input.files[0]) {
      const reader = new FileReader();
      reader.onload = function (e) {
        const textArea = document.getElementById("text") as HTMLTextAreaElement;
        textArea.value = e.target.result.toString();
        highlightSyntax(); // Call highlightSyntax when file is loaded
      };
      reader.readAsText(input.files[0]);
    }
  }
  function clearFields() {
    const titleInput = document.getElementById("title") as HTMLInputElement;
    const textArea = document.getElementById("text") as HTMLTextAreaElement;
    const fileInput = document.getElementById("file") as HTMLInputElement;
    const pasteExpirationSelect = document.getElementById(
      "paste_expiration"
    ) as HTMLSelectElement;
    const codeBlock = document.getElementById("code-block") as HTMLElement;

    titleInput.value = "";
    textArea.value = "";
    fileInput.value = "";
    pasteExpirationSelect.value = "1 minute"; // Reset the dropdown to "1 minute"
    codeBlock.innerHTML = ""; // Clear the highlighted text
    textArea.style.display = "block"; // Show the textarea
    codeBlock.style.display = "none"; // Hide the code block
  }

  function highlightSyntax() {
    const textArea = document.getElementById("text") as HTMLTextAreaElement;
    const codeBlock = document.getElementById("code-block") as HTMLElement;

    codeBlock.style.display = "block";
    textArea.style.display = "none";
    codeBlock.innerHTML = `<pre class="language-${selectedLanguage}"><code>${Prism.highlight(textArea.value, Prism.languages[selectedLanguage], selectedLanguage)}</code></pre>`;
  }
  function editCode() {
    const textArea = document.getElementById("text") as HTMLTextAreaElement;
    const codeBlock = document.getElementById("code-block") as HTMLElement;
    textArea.style.display = "block"; // Show the textarea for editing
    codeBlock.style.display = "none"; // Hide the highlighted code
  }
  function handleLanguageChange(event) {
    selectedLanguage = event.target.value;
    highlightSyntax();
  }
</script>

<div
  class="min-h-screen bg-gray-900 text-white flex items-center justify-center"
>
  <div class="max-w-4xl w-full">
  

    <div class="bg-gray-800 rounded-lg shadow-lg p-8 mb-8">
      <h2 class="text-2xl font-bold mb-4">Create a New Paste</h2>
      <form method="POST" action="?/createPaste" class="space-y-4">
        <div>
          <label for="title" class="block font-medium mb-2">Title</label>
          <input
            type="text"
            id="title"
            name="title"
            placeholder="Enter a title"
            class="bg-gray-700 text-white rounded-md px-4 py-2 w-full focus:outline-none focus:ring-2 focus:ring-indigo-500"
            required
          />
          <div class="text-black p-4">
        
          <select class="bg-indigo-600 hover:bg-indigo-700 text-white font-medium py-2 px-4 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500 m" id="language-select" bind:value={selectedLanguage} on:change={handleLanguageChange}>
              
            <option value="plaintext">Plain Text</option>
            <!-- <option value="javascript">JavaScript</option> -->
              <option value="css">CSS</option>
              <option value="markup">HTML</option>
              <option value="javascript">javascript</option>
             
            </select>
          </div>
        </div>

        <div>
          <label for="file" class="block font-medium mb-2">Select File</label>
          <input
          type="file"
          id="file"
          name="file"
          class="bg-gray-700 text-white rounded-md px-4 py-2 w-full focus:outline-none focus:ring-2 focus:ring-indigo-500"
          accept=".txt,.c,.cpp,.js,.css,.html,.py,.r,.m,.go,.java,.rb,.swift,.sql,.tsx,.sh,.csharp"
          on:change={handleFileSelection}
        />
        
        </div>

        <div>
          <label for="text" class="block font-medium mb-2">Text</label>
          <textarea
            id="text"
            name="text"
            rows="5"
            placeholder="Enter your text here"
            class="bg-gray-700 text-white rounded-md px-4 py-2 w-full focus:outline-none focus:ring-2 focus:ring-indigo-500"
            required
          ></textarea>
          <!-- svelte-ignore a11y-click-events-have-key-events -->
          <div
            id="code-block"
            class="bg-gray-700 text-white rounded-md px-4 py-2 w-full"
            style="display:none;"
            on:click={editCode}
          ></div>
        </div>
        <div>
          <label for="paste_expiration" class="block font-medium mb-2"
            >Paste Expiration</label
          >
          <select
  id="paste_expiration"
  name="paste_expiration"
  class="bg-gray-700 text-white rounded-md px-4 py-2 w-full focus:outline-none focus:ring-2 focus:ring-indigo-500"
>
  <option value="1 minute">1 Minute</option>
  <option value="5 minutes">5 Minutes</option>
  <option value="10 minutes">10 Minutes</option>
  <option value="1 hour">1 Hour</option>
  <option value="1 day">1 Day</option>
  <option value="1 week">1 Week</option>
  <option value="1 month">1 Month</option>
  <option value="2 months">2 Months</option>
  <option value="6 months">6 Months</option>
  <option value="12 months">12 Months</option>
</select>

        </div>
        <button
          type="submit"
          class="bg-green-600 hover:bg-lime-600 text-white font-medium py-2 px-4 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500"
        >
          Submit
        </button>
        <button
          type="button"
          class="bg-red-600 hover:bg-red-700 text-white font-medium py-2 px-4 rounded-md focus:outline-none focus:ring-2 focus:ring-red-500"
          on:click={clearFields}
        >
          Clear
        </button>
      </form>
    </div>


    
    <div class="bg-gray-800 rounded-lg shadow-lg p-8">
      <h2 class="text-2xl font-bold mb-4">Recent Pastes</h2>
      {#if data.pastes && data.pastes.length > 0}
        <ul class="space-y-4">
          {#each data.pastes as paste}
            <li class="bg-gray-700 rounded-lg p-4">
              <h3 class="text-xl font-bold mb-2">{paste.title}</h3>
              <p class="mb-2">{paste.text}</p>
              <p class="text-sm text-gray-400">
                Expires in {formatExpirationTime(paste.paste_expiration)}
              </p>
              <a href={`/pastebin/${paste.id}`} class="text-indigo-400"
                >View Paste</a
              >
            </li>
          {/each}
        </ul>
      {:else}
        <p>No pastes found.</p>
      {/if}
    </div>
  </div>
</div>
