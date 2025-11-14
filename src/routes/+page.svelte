<script lang="ts">
  import {flip} from "svelte/animate";
  import {dndzone} from 'svelte-dnd-action';
  import mergePDFs from "$lib/features/core/pdfUtils";

  const flipDurationMs = 150;
  const dropTargetStyle = {
      outline: "2px dashed rgb(37, 99, 235)",
      backgroundColor: "rgb(219, 234, 254)"
  };

  let pdfFiles = $state<PdfFile[]>([]);
  let downloadUrl = $state<string | null>(null)

  function handleFiles(event: Event) {
    event.preventDefault;
    downloadUrl = null;
    const input = event.target as HTMLInputElement;
    if (input.files?.length) {
      pdfFiles = Array.from(input.files).map(file => ({
        id: crypto.randomUUID(),
        file
      }));;
      input.value = "";
    }
  }

  async function handleMerge() {
    try {
      const blob = await mergePDFs(pdfFiles);
      downloadUrl = URL.createObjectURL(blob);
    } catch (error) {
      console.error("Failed to merge PDFs:", error);
      alert("Something went wrong while merging PDFs. Please try again.");
    }
  }

  function removeFile(index: number) {
    resetDownload()
    pdfFiles.splice(index, 1);
    pdfFiles = [...pdfFiles];
  }

  function handleDnD(event: CustomEvent) {
    resetDownload()
    pdfFiles = event.detail.items as PdfFile[];
  }

  function resetDownload() {
    downloadUrl = null;
  }
</script>

<svelte:head>
  <title>PDF Merger App</title>
</svelte:head>

<main class="min-h-screen bg-gray-100 flex flex-col items-center p-6">
  <!-- Header -->
  <header class="mb-6 text-center">
    <h1 class="text-3xl font-bold">PDF Merge App</h1>
    <p class="text-gray-600 mt-2">Combine multiple PDFs into one file locally.</p>
  </header>

  <!-- Upload Section -->
  <section class="bg-white shadow-md rounded-lg p-6 w-full max-w-lg" aria-labelledby="upload-title">
    <form class="space-y-4" onsubmit={handleMerge}>
      <label id="upload-title" for="uploadPdf" class="block text-xl font-semibold mb-4">
        Upload PDF files
      </label>
      <input 
        id="uploadPdf" 
        type="file" 
        accept="application/pdf" 
        multiple
        onchange={handleFiles}
        class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 
              file:rounded-full file:border-0 file:text-sm file:font-semibold 
              file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100"
      >

      <!-- PDF List -->
      {#if pdfFiles.length > 0}
        <ul
          use:dndzone={{ items: pdfFiles, flipDurationMs, dropTargetStyle }}
          onconsider={handleDnD}
          onfinalize={handleDnD}
          class="mt-4 space-y-2" 
          aria-label="Uploaded files"
        >
          {#each pdfFiles as item, i (item.id)}
            <li 
              animate:flip="{{ duration: flipDurationMs }}"
              class="flex justify-between items-center bg-gray-50 px-4 py-2 rounded-lg"
            >
              <span class="truncate">{i+1}. {item.file.name}</span>
              <button
                type="button"
                onclick={() => removeFile(i)}
                class="text-red-500 hover:text-red-700 text-sm"
                aria-label="Remove {item.file.name}"
              >
                Remove
              </button>
            </li>
          {/each}
        </ul>
      {/if}

      <!-- Merge Button -->
      {#if pdfFiles.length > 1}
        <button
          type="submit"
          class="mt-5 w-full bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700"
        >
          Merge
        </button>
        <p class="text-sm text-gray-500">
          All processing happens locally in your browser.
        </p>
      {/if}
    </form>

    <!-- Result Download link -->
    {#if downloadUrl}
      <a
        href={downloadUrl}
        download="merged.pdf"
        class="mt-4 block text-center text-blue-600 hover:underline"
      >
        Download Merged PDF
      </a>
    {/if}
  </section>
</main>