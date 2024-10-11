<script lang="ts">
  import { page } from "$app/stores";
  import AuthCheck from "$lib/components/AuthCheck.svelte";
  import { db, userData, user } from "$lib/firebase";
  import { doc, updateDoc } from "firebase/firestore";
  import { writable } from "svelte/store";

  const platforms = ["Twitter", "YouTube", "LinkedIn", "GitHub"];
  const formDefaults = {
    platform: "Twitter",
    label: "",
    url: "https://",
  };
  const formData = writable(formDefaults);

  let links = writable([]);
  let errors = writable({});

  $: if ($userData) {
    links.set($userData.links || []);
  }

  function addNewLink() {
    links.update(currentLinks => [...currentLinks, { ...formDefaults, id: Date.now().toString() }]);
  }

  function removeLink(id: string) {
    links.update(currentLinks => currentLinks.filter(link => link.id !== id));
    errors.update(currentErrors => {
      delete currentErrors[id];
      return currentErrors;
    });
  }

  function updateLink(id: string, field: string, value: string) {
    links.update(currentLinks => 
      currentLinks.map(link => 
        link.id === id ? { ...link, [field]: value } : link
      )
    );
    validateLink(id);
  }

  function validateLink(id: string) {
    const link = $links.find(link => link.id === id);
    if (!link) return;

    let linkErrors = {};

    if (!link.url) {
      linkErrors['url'] = 'URL is required';
    } else {
      const urlPattern = getUrlPattern(link.platform);
      if (!urlPattern.test(link.url)) {
        linkErrors['url'] = `Invalid ${link.platform} URL`;
      }
    }

    errors.update(currentErrors => ({
      ...currentErrors,
      [id]: linkErrors
    }));
  }

  function getUrlPattern(platform: string) {
    switch (platform) {
      case 'Twitter':
      return /^https?:\/\/(www\.)?(twitter\.com|x\.com)\/[a-zA-Z0-9_]{1,15}\/?$/;
      case 'YouTube':
        return /^https?:\/\/(www\.)?youtube\.com\/(channel\/UC[\w-]{21}[AQgw]|(c\/|user\/)?[\w-]+)$/;
      case 'LinkedIn':
        return /^https?:\/\/(www\.)?linkedin\.com\/(in|company)\/[\w-]+\/?$/;
      case 'GitHub':
        return /^https?:\/\/(www\.)?github\.com\/[a-zA-Z0-9_-]+\/?$/;
      default:
        return /.+/; // Allow any non-empty string if platform is not recognized
    }
  }

  async function saveLinks() {
    $links.forEach(link => validateLink(link.id));
    
    if (Object.values($errors).some(error => Object.keys(error).length > 0)) {
      alert('Please correct the errors before saving.');
      return;
    }

    const userRef = doc(db, "users", $user!.uid);
    await updateDoc(userRef, { links: $links });
    alert('Links saved successfully!');
  }

  let draggedItem: number;

  function handleDragStart(e: DragEvent, index: number) {
    draggedItem = index;
    e.dataTransfer!.effectAllowed = 'move';
    e.dataTransfer!.setData('text/plain', index.toString());
    (e.target as HTMLElement).style.opacity = '0.4';
  }

  function handleDragOver(e: DragEvent, index: number) {
    e.preventDefault();
    (e.target as HTMLElement).classList.add('drag-over');
  }

  function handleDragLeave(e: DragEvent) {
    (e.target as HTMLElement).classList.remove('drag-over');
  }

  function handleDrop(e: DragEvent, index: number) {
    e.preventDefault();
    (e.target as HTMLElement).classList.remove('drag-over');
    const newIndex = index;
    if (newIndex !== draggedItem) {
      links.update(items => {
        const item = items[draggedItem];
        items.splice(draggedItem, 1);
        items.splice(newIndex, 0, item);
        return items;
      });
    }
  }

  function handleDragEnd(e: DragEvent) {
    (e.target as HTMLElement).style.opacity = '1';
  }
</script>

<AuthCheck>
  <div class="flex flex-col md:flex-row max-w-6xl mx-auto mt-8 gap-8">
    <!-- Left side: Mobile preview (only visible on desktop) -->
    <div class="hidden md:block w-1/3">
      <div class="bg-gray-900 p-8 rounded-3xl shadow-lg flex justify-center items-center">
        <div class="bg-gray-800 p-4 rounded-3xl shadow-md border-4 border-gray-600" style="width: 37rem; height: 667px; overflow-y: auto;">
          <div class="flex flex-col items-center justify-center h-full">
            <img
              src={$userData?.photoURL || "/user.png"}
              alt="Profile"
              class="w-24 h-24 rounded-full mb-4"
            />
            <h2 class="text-xl font-bold text-neutral-100">{$userData?.firstName} {$userData?.lastName}</h2>
            <p class="text-center mt-2 text-neutral-300">{$userData?.bio}</p>
            <ul class="mt-4 w-full max-w-[300px]">
              {#each $links as link}
                <li class="bg-neutral-700 p-2 rounded mb-2 text-neutral-100 text-center">
                  {link.platform}: {link.label}
                </li>
              {/each}
            </ul>
          </div>
        </div>
      </div>
    </div>

    <!-- Right side: Edit panel -->
    <div class="w-full md:w-2/3">
      <div class="bg-gray-800 p-6 rounded-lg shadow-md">
        <h2 class="text-2xl font-bold mb-4 text-neutral-100">Customize Your Links</h2>
        <p class="mb-4 text-neutral-300">Add or remove links to your profile</p>
        
        <button on:click={addNewLink} class="btn btn-primary mb-4">Add New Link</button>

        {#each $links as link, index (link.id)}
          <div
            class="card bg-gray-700 shadow-xl mb-4 cursor-move"
            draggable="true"
            on:dragstart={(e) => handleDragStart(e, index)}
            on:dragover={(e) => handleDragOver(e, index)}
            on:dragleave={handleDragLeave}
            on:drop={(e) => handleDrop(e, index)}
            on:dragend={handleDragEnd}
          >
            <div class="card-body">
              <div class="flex justify-between items-center">
                <span class="text-neutral-100">Link #{index + 1}</span>
                <button on:click={() => removeLink(link.id)} class="btn btn-sm btn-error">Remove</button>
              </div>
              <label class="label text-neutral-300">Platform</label>
              <select 
                bind:value={link.platform} 
                on:change={(e) => updateLink(link.id, 'platform', e.target.value)}
                class="select select-bordered w-full bg-neutral-600 text-neutral-100"
              >
                {#each platforms as platform}
                  <option value={platform}>{platform}</option>
                {/each}
              </select>
              <label class="label text-neutral-300">Link Label</label>
              <input 
                type="text" 
                bind:value={link.label}
                on:input={(e) => updateLink(link.id, 'label', e.target.value)}
                placeholder="Enter link label" 
                class="input input-bordered w-full bg-neutral-600 text-neutral-100"
              />
              <label class="label text-neutral-300">URL</label>
              <input 
                type="url" 
                bind:value={link.url}
                on:input={(e) => updateLink(link.id, 'url', e.target.value)}
                placeholder="https://" 
                class="input input-bordered w-full bg-neutral-600 text-neutral-100"
              />
              {#if $errors[link.id] && $errors[link.id].url}
                <p class="text-red-500 mt-1">{$errors[link.id].url}</p>
              {/if}
            </div>
          </div>
        {/each}

        <div class="text-right">
          <button on:click={saveLinks} class="btn btn-success">Save</button>
        </div>
      </div>
    </div>
  </div>
</AuthCheck>

<style>
  .drag-over {
    border: 2px dashed #4a5568;
  }
</style>