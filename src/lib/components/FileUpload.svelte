<script>
  import { writable } from 'svelte/store';

  const MAX_FILES = 10;
  let files = writable([]);
  let downloadUrl = writable(null);
  let downloadType = writable(null);
  let selectedOption = writable('face'); // 선택된 옵션을 저장할 writable 스토어
  let processing = writable(false); // 파일 업로드 처리 상태

  function preventDefaults(e) {
    console.log('preventDefaults called'); // 디버깅 로그
    e.preventDefault();
    e.stopPropagation();
  }

  function highlight(e) {
    console.log('highlight called'); // 디버깅 로그
    e.currentTarget.classList.add('bg-gray-100');
  }

  function unhighlight(e) {
    console.log('unhighlight called'); // 디버깅 로그
    e.currentTarget.classList.remove('bg-gray-100');
  }

  function handleFiles(selectedFiles) {
    console.log('handleFiles called', selectedFiles); // 디버깅 로그
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
    console.log('handleDrop called'); // 디버깅 로그
    unhighlight(e);
    let dt = e.dataTransfer;
    let droppedFiles = Array.from(dt.files);

    handleFiles(droppedFiles);
  }

  function handleChange(e) {
    console.log('handleChange called'); // 디버깅 로그
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
    console.log('startMosaicing function called'); // 디버깅 로그
    processing.set(true); // 처리 상태 설정
    
    const formData = new FormData();
    let currentFiles;

    files.subscribe(value => currentFiles = value)();
    console.log('Files:', currentFiles);

    currentFiles.forEach(file => {
      formData.append('files', file);
    });

    let selectedOptionValue;
    selectedOption.subscribe(value => selectedOptionValue = value)();
    console.log('Selected option:', selectedOptionValue);

    let url;
    if (selectedOptionValue === 'face') {
      url = 'http://34.22.109.229:8000/face/images/upload';
    } else if (selectedOptionValue === 'car_plate') {
      url = 'http://34.22.109.229:8000/car_plate/images/upload';
    }

    console.log('POST URL:', url);

    try {
      const response = await fetch('https://www.nu-eta.shop/face/images/upload', {
        method: 'POST',
        body: formData
      });

      console.log('Fetch response:', response);

      if (!response.ok) {
        throw new Error('Network response was not ok');
      }

      const blob = await response.blob();
      const downloadLink = URL.createObjectURL(blob);
      downloadUrl.set(downloadLink);

      if (blob.type === 'application/zip') {
        downloadType.set('zip');
      } else if (blob.type.startsWith('image/')) {
        downloadType.set('image');
      }

      // 업로드 성공 시 파일 목록 초기화
      files.set([]);
    } catch (error) {
      console.error('There was a problem with the fetch operation:', error);
    } finally {
      processing.set(false); // 처리 상태 해제
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

    <!-- 라디오 버튼 추가 -->
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
