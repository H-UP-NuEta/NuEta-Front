<script>
  const MAX_FILES = 10;
  let files = [];

  function preventDefaults(e) {
    e.preventDefault();
    e.stopPropagation();
  }

  function highlight(e) {
    e.target.classList.add('bg-gray-100');
  }

  function unhighlight(e) {
    e.target.classList.remove('bg-gray-100');
  }

  function handleFiles(selectedFiles) {
    for (let file of selectedFiles) {
      if (files.length < MAX_FILES && (file.type.startsWith('image/') || file.type.startsWith('video/'))) {
        files = [...files, file];
      } else {
        break; // Stop adding files if we reach the maximum number
      }
    }
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
</script>

<section class="my-8"
    on:dragover|preventDefault={highlight}
    on:dragenter|preventDefault={highlight}
    on:dragleave|preventDefault={unhighlight}
    on:drop|preventDefault={handleDrop}>

  <div class="flex flex-col items-center py-12 border-2 border-dashed border-gray-300 hover:border-gray-400 transition-colors">
    {#if files.length > 0}
      <div class="p-4">
        <h3 class="font-bold text-lg">Selected Files</h3>
        <ul class="list-disc">
          {#each files as file, index}
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
  
  <button class="mt-4 bg-black text-white py-2 px-6 rounded shadow-lg hover:bg-gray-700 transition-colors">
    Start Mosaicing
  </button>
</section>
