<div
    class="relative border-2 border-dashed border-gray-300 rounded-lg p-6 hover:border-pink-300 transition-colors"
    class:border-pink-400={isDragging}
    on:dragenter={handleDragEnter}
    on:dragleave={handleDragLeave}
    on:dragover|preventDefault
    on:drop|preventDefault={handleDrop}
>
    <input
        type="file"
        accept="image/*"
        class="hidden"
        bind:this={fileInput}
        on:change={handleFileSelect}
    />
    
    <div class="flex flex-col items-center justify-center gap-2">
        <svg class="w-12 h-12 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
        </svg>
        <p class="text-sm text-gray-500">
            Drag and drop an image here, or
            <button
                class="text-pink-500 hover:text-pink-600 font-medium"
                on:click={() => fileInput?.click()}
            >
                browse
            </button>
        </p>
    </div>
</div>

<script lang="ts">
    import { createEventDispatcher } from 'svelte';
    
    const dispatch = createEventDispatcher<{
        upload: { file: File };
    }>();
    
    let isDragging = false;
    let fileInput: HTMLInputElement;
    
    function handleDragEnter(event: DragEvent) {
        event.preventDefault();
        isDragging = true;
    }
    
    function handleDragLeave(event: DragEvent) {
        event.preventDefault();
        isDragging = false;
    }
    
    function handleDrop(event: DragEvent) {
        event.preventDefault();
        isDragging = false;
        
        const file = event.dataTransfer?.files[0];
        if (file && file.type.startsWith('image/')) {
            dispatch('upload', { file });
        }
    }
    
    function handleFileSelect(event: Event) {
        const input = event.target as HTMLInputElement;
        const file = input.files?.[0];
        
        if (file && file.type.startsWith('image/')) {
            dispatch('upload', { file });
        }
    }
</script> 