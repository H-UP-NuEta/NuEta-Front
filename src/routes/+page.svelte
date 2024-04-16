<script>
  import Header from '$lib/components/Header.svelte';
  import FileUpload from '../lib/components/FileUpload.svelte';
</script>

<main>
  <Header />
  <FileUpload />
  
</main>

<style>
  main {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
    box-sizing: border-box;
  }
  
  /* Responsive adjustments */
  @media (max-width: 768px) {
    main {
      padding: 10px;
    }
  }
</style>
