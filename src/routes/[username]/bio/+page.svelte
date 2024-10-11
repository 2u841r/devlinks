<script lang="ts">
  import AuthCheck from "$lib/components/AuthCheck.svelte";
  import { user, userData, storage, db } from "$lib/firebase";
  import { getDownloadURL, ref, uploadBytes } from "firebase/storage";
  import { enhance } from "$app/forms";
  import type { PageData } from "./$types";

  export let data: PageData;

  let previewURL: string;
  let uploading = false;
  let firstName = data.firstName || "";
  let lastName = data.lastName || "";
  let bio = data.bio || "";

  $: if ($userData) {
    previewURL = $userData.photoURL || "/user.png";
  }

  async function upload(e: any) {
    uploading = true;
    const file = e.target.files[0];
    previewURL = URL.createObjectURL(file);
    const storageRef = ref(storage, `users/${$user!.uid}/profile.png`);
    const result = await uploadBytes(storageRef, file);
    const url = await getDownloadURL(result.ref);

    await fetch('/api/updatePhotoURL', {
      method: 'POST',
      body: JSON.stringify({ photoURL: url }),
      headers: {
        'Content-Type': 'application/json'
      }
    });

    uploading = false;
  }
</script>

<AuthCheck>
  <div class="flex flex-col md:flex-row max-w-6xl mx-auto mt-8 gap-8">
    <!-- Left side: Mobile preview (only visible on desktop) -->
    <div class="hidden md:block w-1/3">
      <div class="bg-gray-900 p-8 rounded-3xl shadow-lg flex justify-center items-center">
        <div class="bg-gray-800 p-4 rounded-3xl shadow-md border-4 border-gray-600" style="width: 30rem; height: 667px; overflow-y: auto;">
          <div class="flex flex-col items-center justify-center h-full">
            <img
              src={previewURL}
              alt="Profile"
              class="w-24 h-24 rounded-full mb-4"
            />
            <h2 class="text-xl font-bold text-white">{firstName} {lastName}</h2>
            <p class="text-center mt-2 text-gray-300">{bio}</p>
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
              <h3 class="text-xl font-semibold mb-2 text-white">Profile Picture</h3>
              <div class="flex items-center justify-between">
                <span class="text-white">Profile Picture</span>
                <img
                  src={previewURL}
                  alt="Profile"
                  class="w-16 h-16 rounded-full"
                />
                <div>
                  <p class="text-sm text-gray-300 mb-2">Image must be below 1024x1024.</p>
                  <p class="text-sm text-gray-300">Use PNG or JPG format.</p>
                  <label for="photoURL" class="btn btn-sm btn-outline btn-primary mt-2">
                    {uploading ? "Uploading..." : "Change Photo"}
                  </label>
                  <input
                    id="photoURL"
                    on:change={upload}
                    type="file"
                    class="hidden"
                    accept="image/png, image/jpeg"
                  />
                </div>
              </div>
            </div>
          </div>

          <div class="card bg-gray-700 shadow-xl mb-4">
            <div class="card-body">
              <h3 class="text-xl font-semibold mb-2 text-white">Profile Details</h3>
              <div class="mb-4">
                <label for="firstName" class="block text-sm font-medium text-gray-300">First Name</label>
                <input name="firstName" type="text" id="firstName" bind:value={firstName} class="mt-1 p-2 block w-full rounded-md bg-gray-600 border-gray-500 text-white shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50">
              </div>
              <div class="mb-4">
                <label for="lastName" class="block text-sm font-medium text-gray-300">Last Name</label>
                <input name="lastName" type="text" id="lastName" bind:value={lastName} class="mt-1 p-2 block w-full rounded-md bg-gray-600 border-gray-500 text-white shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50">
              </div>
              <div>
                <label for="bio" class="block text-sm font-medium text-gray-300">Bio</label>
                <textarea name="bio" id="bio" bind:value={bio} rows="3" class="mt-1 p-2 block w-full rounded-md bg-gray-600 border-gray-500 text-white shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"></textarea>
              </div>
            </div>
          </div>
        </div>
        <div class="text-right mt-4">
          <button type="submit" class="btn btn-primary">Save</button>
        </div>
      </form>
    </div>
  </div>
</AuthCheck>

<style>
  :global(body) {
    background-color: #1f2937;
    color: #ffffff;
  }
</style>