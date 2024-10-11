<script lang="ts">
  import { page } from "$app/stores";
  import AuthCheck from "$lib/components/AuthCheck.svelte";
  import { db, userData, user } from "$lib/firebase";
  import { doc, updateDoc, arrayUnion, arrayRemove } from "firebase/firestore";
  import { writable } from "svelte/store";

  const platforms = ["Twitter", "YouTube", "LinkedIn", "GitHub"];
  const formDefaults = {
    platform: "Twitter",
    label: "",
    url: "https://",
  };
  const formData = writable(formDefaults);

  let links = writable([]);

  $: if ($userData) {
    links.set($userData.links || []);
  }

  function addNewLink() {
    links.update((currentLinks) => [
      ...currentLinks,
      { ...formDefaults, id: Date.now().toString() },
    ]);
  }

  function removeLink(id: string) {
    links.update((currentLinks) =>
      currentLinks.filter((link) => link.id !== id)
    );
  }

  function updateLink(id: string, field: string, value: string) {
    links.update((currentLinks) =>
      currentLinks.map((link) =>
        link.id === id ? { ...link, [field]: value } : link
      )
    );
  }

  async function saveLinks() {
    const userRef = doc(db, "users", $user!.uid);
    await updateDoc(userRef, { links: $links });
  }
</script>

<AuthCheck>
  <div class="flex flex-col md:flex-row max-w-6xl mx-auto mt-8 gap-8">
    <!-- Left side: Mobile preview (only visible on desktop) -->
    <div class="hidden md:block w-1/3">
      <div
        class="bg-gray-900 p-8 rounded-3xl shadow-lg flex justify-center items-center"
      >
        <div
          class="bg-gray-800 p-4 rounded-3xl shadow-md border-4 border-gray-600"
          style="width: 375px; height: 667px; overflow-y: auto;"
        >
          <div class="flex flex-col items-center justify-center h-full">
            <img
              src={$userData?.photoURL || "/user.png"}
              alt="Profile"
              class="w-24 h-24 rounded-full mb-4"
            />
            <h2 class="text-xl font-bold text-neutral-100">
              {$userData?.firstName}
              {$userData?.lastName}
            </h2>
            <p class="text-center mt-2 text-neutral-300">{$userData?.bio}</p>
            <ul class="mt-4 w-full max-w-[300px]">
              {#each $links as link}
                <li
                  class="bg-neutral-700 p-2 rounded mb-2 text-neutral-100 text-center"
                >
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
      <form method="POST" use:enhance>
        <div class="bg-gray-800 p-6 rounded-lg shadow-md">
          <div class="card bg-gray-700 shadow-xl mb-4">
            <div class="card-body">
              <h3 class="text-xl font-semibold mb-2 text-white">
                Customize Your Links
              </h3>
              <p class="text-sm text-gray-300 mb-4">
                Add or remove links to your profile
              </p>

              <button
                on:click={addNewLink}
                class="btn btn-sm btn-outline btn-primary mb-4"
                >Add New Link</button
              >

              {#each $links as link (link.id)}
                <div class="card bg-gray-600 shadow-xl mb-4">
                  <div class="card-body">
                    <div class="flex justify-between items-center mb-4">
                      <span class="text-white"
                        >Link #{$links.indexOf(link) + 1}</span
                      >
                      <button
                        on:click={() => removeLink(link.id)}
                        class="btn btn-sm btn-error">Remove</button
                      >
                    </div>

                    <div class="mb-4">
                      <label class="block text-sm font-medium text-gray-300"
                        >Platform</label
                      >
                      <select
                        bind:value={link.platform}
                        on:change={(e) =>
                          updateLink(link.id, "platform", e.target.value)}
                        class="mt-1 p-2 block w-full rounded-md bg-gray-600 border-gray-500 text-white shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                      >
                        {#each platforms as platform}
                          <option value={platform}>{platform}</option>
                        {/each}
                      </select>
                    </div>

                    <div class="mb-4">
                      <label class="block text-sm font-medium text-gray-300"
                        >Link Label</label
                      >
                      <input
                        type="text"
                        bind:value={link.label}
                        on:input={(e) =>
                          updateLink(link.id, "label", e.target.value)}
                        placeholder="Enter link label"
                        class="mt-1 p-2 block w-full rounded-md bg-gray-600 border-gray-500 text-white shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                      />
                    </div>

                    <div>
                      <label class="block text-sm font-medium text-gray-300"
                        >URL</label
                      >
                      <input
                        type="url"
                        bind:value={link.url}
                        on:input={(e) =>
                          updateLink(link.id, "url", e.target.value)}
                        placeholder="https://"
                        class="mt-1 p-2 block w-full rounded-md bg-gray-600 border-gray-500 text-white shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
                      />
                    </div>
                  </div>
                </div>
              {/each}
            </div>
          </div>
        </div>
        <div class="text-right mt-4">
          <button type="submit" class="btn btn-primary" on:click={saveLinks}
            >Save Changes</button
          >
        </div>
      </form>
    </div>
  </div>
</AuthCheck>

<style>
  :global(body) {
    background-color: hsl(var(--n));
    color: hsl(var(--nc));
  }
</style>
