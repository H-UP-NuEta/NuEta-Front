<script>
  import { writable } from 'svelte/store';

  const MAX_FILES = 10;
  const MAX_FILE_SIZE_MB = 10;  // Set the max file size limit (in MB)
  const MAX_FILE_SIZE_BYTES = MAX_FILE_SIZE_MB * 1024 * 1024;
  let files = writable([]);
  let downloadUrl = writable(null);
  let downloadType = writable(null);
  let selectedOption = writable('face');
  let processing = writable(false);

  function preventDefaults(e) {
    e.preventDefault();
    e.stopPropagation();
  }

  function highlight(e) {
    e.currentTarget.classList.add('bg-gray-100');
  }

  function unhighlight(e) {
    e.currentTarget.classList.remove('bg-gray-100');
  }

  function handleFiles(selectedFiles) {
    files.update(currentFiles => {
      let newFiles = [...currentFiles];
      selectedFiles.forEach(file => {
        if (newFiles.length < MAX_FILES && (file.type.startsWith('image/') || file.type.startsWith('video/')) && file.size <= MAX_FILE_SIZE_BYTES) {
          newFiles.push(file);
        }
      });
      return newFiles;
    });
  }

  function handleDrop(e) {
    unhighlight(e);
    let dt = e.dataTransfer;
    let droppedFiles = Array.from(dt.files);
    handleFiles(droppedFiles);
  }

  function handleChange(e) {
    let selectedFiles = Array.from(e.target.files);
    handleFiles(selectedFiles);
  }

  function removeFile(index) {
    files.update(currentFiles => {
      const newFiles = [...currentFiles];
      newFiles.splice(index, 1);
      return newFiles;
    });
  }

  async function startMosaicing() {
    processing.set(true);
    const formData = new FormData();
    let currentFiles;
    files.subscribe(value => currentFiles = value)();
    currentFiles.forEach(file => formData.append('files', file));
    let selectedOptionValue;
    selectedOption.subscribe(value => selectedOptionValue = value)();

    let url = selectedOptionValue === 'face'
      ? 'https://www.nu-eta.shop/face/images/upload'
      : 'https://www.nu-eta.shop/car_plate/images/upload';

    try {
      const response = await fetch(url, { method: 'POST', body: formData });
      if (!response.ok) throw new Error('Network response was not ok');

      const blob = await response.blob();
      const downloadLink = URL.createObjectURL(blob);
      downloadUrl.set(downloadLink);
      downloadType.set(blob.type === 'application/zip' ? 'zip' : 'image');
      files.set([]);
    } catch (error) {
      console.error('Fetch operation error:', error);
    } finally {
      processing.set(false);
    }
  }
</script>

<section class="my-8"
    aria-label="File Upload Section"
    on:dragover|preventDefault={highlight}
    on:dragenter|preventDefault={highlight}
    on:dragleave|preventDefault={unhighlight}
    on:drop|preventDefault={handleDrop}>

  <div class="flex flex-col items-center py-12 border-2 border-dashed border-gray-300 hover:border-gray-400 transition-colors">
    {#if $files.length > 0}
      <div class="p-4">
        <h3 class="font-bold text-lg">Selected Files</h3>
        <ul class="list-disc">
          {#each $files as file, index}
            <li class="text-sm text-gray-600 mt-2 flex items-center">
              {index + 1}: {file.name} - {file.size} bytes
              <button 
                class="ml-2 text-red-500 hover:text-red-700" 
                on:click={() => removeFile(index)}>
                <i class="fas fa-times"></i>
              </button>
            </li>
          {/each}
        </ul>
      </div>
    {/if}

    <div class="my-4">
      <label class="block text-sm font-medium text-gray-700">Select Upload Option</label>
      <div class="mt-2 space-y-2">
        <label class="inline-flex items-center">
          <input type="radio" name="uploadOption" value="face" bind:group={$selectedOption} checked>
          <span class="ml-2">Face</span>
        </label>
        <label class="inline-flex items-center ml-4">
          <input type="radio" name="uploadOption" value="car_plate" bind:group={$selectedOption}>
          <span class="ml-2">Car Plate</span>
        </label>
      </div>
    </div>

    <label for="file-upload" class="cursor-pointer text-center space-y-2">
      <div class="flex flex-col items-center justify-center space-y-1">
        <div class="text-gray-400 text-4xl">
          <i class="fas fa-file-upload"></i>
        </div>
        <span class="text-gray-500">Drag and drop your files here or</span>
        <span class="text-blue-600 hover:text-blue-800">Choose Files</span>
      </div>
    </label>
  </div>
  
  <input
    type="file"
    multiple
    id="file-upload"
    class="hidden"
    on:change={handleChange}
  />
  
  <button 
    class="mt-4 bg-black text-white py-2 px-6 rounded shadow-lg hover:bg-gray-700 transition-colors"
    on:click={startMosaicing}>
    Start Mosaicing
  </button>
  
  {#if $downloadUrl}
    <a 
      href={$downloadUrl} 
      download={$downloadType === 'zip' ? 'mosaiced_images.zip' : 'mosaiced_image.jpg'}
      class="mt-4 bg-green-500 text-white py-2 px-6 rounded shadow-lg hover:bg-green-700 transition-colors block text-center">
      Download {$downloadType === 'zip' ? 'ZIP File' : 'Image'}
    </a>
  {:else if $processing}
    <div class="mt-4 bg-yellow-500 text-white py-2 px-6 rounded shadow-lg text-center">
      Processing...
    </div>
  {/if}
</section>
