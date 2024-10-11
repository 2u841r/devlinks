<script>
  import { page } from "$app/stores";
  import { userData } from "$lib/firebase";
  import linkIcon from "$lib/svg/link.svg";
  import profileIcon from "$lib/svg/profile.svg";
  $: username = $userData?.username;
  $: currentPath = $page.url.pathname;
  $: isLoggedIn = !!$userData;
</script>

<nav
  class="flex justify-between items-center p-4 bg-neutral border-b border-gray-300"
>
  <!-- Left: Logo and Text -->
  <a href="/" class="flex items-center">
    <img src="/logo2.png" alt="Logo" class="h-8 mr-2" />
    <span class="text-lg font-bold hidden sm:inline">devlinks</span>
  </a>

  <!-- Center: Navigation Buttons or Create Your Link Button -->
  <div class="flex space-x-4">
    {#if isLoggedIn}
      <a
        href="/{username}/edit"
        class="py-2 px-4 rounded-lg transition-colors duration-200 flex items-center space-x-2
        {currentPath.endsWith('/edit')
          ? 'bg-[#EDEAFE] text-[#623BFE]'
          : 'text-white'}"
      >
        <img
          src={linkIcon}
          alt="Links"
          class="h-5 w-5"
          style="filter: {currentPath.endsWith('/edit') ? 'invert(27%) sepia(82%) saturate(1738%) hue-rotate(230deg) brightness(97%) contrast(96%)' : 'invert(1)'}"
        />
        <span class="hidden sm:inline">Links</span>
      </a>

      <a
        href="/{username}/bio"
        class="py-2 px-4 rounded-lg transition-colors duration-200 flex items-center space-x-2
        {currentPath.endsWith('/bio')
          ? 'bg-[#EDEAFE] text-[#623BFE]'
          : 'text-white'}"
      >
        <img
          src={profileIcon}
          alt="Profile"
          class="h-5 w-5"
          style="filter: {currentPath.endsWith('/bio') ? 'invert(27%) sepia(82%) saturate(1738%) hue-rotate(230deg) brightness(97%) contrast(96%)' : 'invert(1)'}"
        />
        <span class="hidden sm:inline">Profile Details</span>
      </a>
    {:else}
      <a
        href="/login"
        class="py-2 px-4 rounded-lg transition-colors duration-200 bg-[#623BFE] text-white hover:bg-[#4e2fd9]"
      >
        Create your link
      </a>
    {/if}
  </div>

  <!-- Right: Preview Button -->
  <div>
    {#if isLoggedIn}
      <a
        href="/{username}"
        class="bg-blue-500 hover:bg-blue-700 text-white py-2 px-4 rounded"
      >
        Preview
      </a>
    {:else}
      <a
        href="/login"
        class="bg-blue-500 hover:bg-blue-700 text-white py-2 px-4 rounded"
      >
        Login
      </a>
    {/if}
  </div>
</nav>

<style>
  a.isactive {
    border-bottom-color: #007bff;
    font-weight: bold;
  }
  a:not(.isactive) {
    border-bottom-color: transparent;
  }
</style>
