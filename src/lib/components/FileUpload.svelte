<script>
  import { writable } from 'svelte/store';

  const MAX_FILES = 10;
  let files = writable([]);
  let downloadUrl = writable(null);
  let downloadType = writable(null);

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
        if (newFiles.length < MAX_FILES && (file.type.startsWith('image/') || file.type.startsWith('video/'))) {
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

  async function startMosaicing() {
    const formData = new FormData();
    files.update(currentFiles => {
      currentFiles.forEach(file => {
        formData.append('files', file);
      });
      return currentFiles;
    });

    try {
      const response = await fetch('http://34.22.109.229:8000/face/images/upload', {
        method: 'POST',
        body: formData
      });

      if (!response.ok) {
        throw new Error('Network response was not ok');
      }

      const blob = await response.blob();
      const url = URL.createObjectURL(blob);
      downloadUrl.set(url);

      // Determine the type of the file to set the appropriate download type
      if (blob.type === 'application/zip') {
        downloadType.set('zip');
      } else if (blob.type.startsWith('image/')) {
        downloadType.set('image');
      }
    } catch (error) {
      console.error('There was a problem with the fetch operation:', error);
      // Handle error response
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
            <li class="text-sm text-gray-600 mt-2">{index + 1}: {file.name} - {file.size} bytes</li>
          {/each}
        </ul>
      </div>
    {/if}
    
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
  {/if}
</section>
