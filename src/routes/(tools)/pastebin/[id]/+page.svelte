<script>
  import { onMount } from "svelte";
  import { page } from "$app/stores"; // Import $page from $app/stores
  import { HighlightAuto } from "svelte-highlight";
  import atomOneDark from "svelte-highlight/styles/atom-one-dark";
  import ClipboardJS from "clipboard";
  import toast, { Toaster } from "svelte-french-toast";
  import { jsPDF } from "jspdf";
  let id = null;
  let paste = null;
  let code = null;
  const doc = new jsPDF();

  onMount(async () => {
    id = $page.params.id; // Now $page is correctly imported and can be used
    console.log(`fetching /pastebin/api?id=${id}`);
    fetch(`/pastebin/api?id=${id}`)
      .then((res) => res.json())
      .then((data) => {
        // console.log("Fetched paste:", data.id);
        paste = data.id;
        new ClipboardJS(".btn-clip");
      })
      .catch((error) => {
        console.log("Error fetching paste:", error);
      });
  });

  function formatExpirationTime(expirationTimestamp) {
    const secondsRemaining = Math.floor(
      (expirationTimestamp - Date.now()) / 1000
    );

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

  function handleassignment() {
    code = paste.text;

    return "";
  }

  async function shareLink() {
    if (navigator.share) {
      try {
        await navigator.share({
          title: "Check this out!",
          text: "I found this interesting link:",
          url: `/pastebin/${id}`,
        });
        console.log("Link shared successfully");
      } catch (error) {
        console.error("Error sharing:", error);
      }
    } else {
      alert("Web Share API is not supported in your browser.");
    }
  }
</script>

<svelte:head>
  {@html atomOneDark}
</svelte:head>

{#if paste}
  {handleassignment()}
  <div class="container mx-auto my-8">
    <div class="bg-gray-800 rounded-lg shadow-md p-6">
      <div class="card-actions">
        <div
          class="badge badge-outline flex justify-betweenflex justify-between"
        >
          <h1 class="border rounded-lg p-2 text-white text-2xl mb-4">{paste.title}</h1>
         
        </div>

        <HighlightAuto {code} />
      </div>
      <Toaster />
      <div class="flex justify-between">
        <!-- copy button -->
        <button
          on:click={() => {
            // toast("Here is your toast.");
            toast.success("copied to clipboard");
          }}
          class="btn-clip bg-green-600 hover:bg-lime-600 text-white font-medium py-2 px-4 mt-3 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500"
          data-clipboard-text={code}
        >
          COPY TEXT
        </button>
        <div>
          <!-- download pdf button -->
          <button
            on:click={() => {
              doc.text(`${paste.text}`, 10, 10);
              doc.save(`${paste.title}.pdf`);
            }}
            class="btn-clip bg-blue-600 hover:bg-indigo-700 text-white font-medium py-2 px-4 mt-3 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500"
            data-clipboard-text={code}
          >
            <svg
              class="h-5 w-5 text-white"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" />
              <polyline points="7 10 12 15 17 10" />
              <line x1="12" y1="15" x2="12" y2="3" /></svg
            >
          </button>
          <!-- share button -->
          <button
            on:click={shareLink}
            class="btn-clip bg-blue-600 hover:bg-indigo-700 text-white font-medium py-2 px-4 mt-3 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500"
          >
            <svg
              class="h-5 w-5 text-white"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <circle cx="18" cy="5" r="3" /> <circle cx="6" cy="12" r="3" />
              <circle cx="18" cy="19" r="3" />
              <line x1="8.59" y1="13.51" x2="15.42" y2="17.49" />
              <line x1="15.41" y1="6.51" x2="8.59" y2="10.49" /></svg
            >
          </button>
        </div>
      </div>
    </div>
  </div>
{:else}
  <div class="container mx-auto my-8">
    <div class="bg-white rounded-lg shadow-md p-6">
      <p>Paste not found or expired.</p>
    </div>
  </div>
{/if}
