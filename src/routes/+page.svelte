<!-- Landing/Splash Page -->

<!-- Main container with gradient background -->
<div class="h-screen w-screen bg-[radial-gradient(ellipse_at_center,_var(--tw-gradient-stops))] from-rose-300 via-pink-300 to-pink-100 text-white p-4 flex flex-col items-center justify-center">
    <!-- Logo/Emoji Display -->
    <h1 class="text-7xl">🎀</h1>
    
    <!-- Image Upload Area -->
    <div class="mt-8 w-full max-w-md">
        <div
            class="flex flex-col items-center justify-center border-2 border-dashed border-white/50 rounded-lg p-6 hover:border-white transition-all duration-300 text-center"
            class:border-white={isDragging}
            class:h-[250px]={!backgroundImage} 
            class:sm:h-[500px]={!backgroundImage}
            style={backgroundImage ? 'height: auto' : ''}
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
            
            {#if backgroundImage}
                <img 
                    src={backgroundImage} 
                    alt="Uploaded preview" 
                    class="w-full max-h-[250px] sm:max-h-[500px] object-contain rounded-lg mb-2 transition-all duration-300"
                />
            {:else}
                <div class="flex flex-col items-center justify-center gap-2">
                    <svg class="w-12 h-12 text-white/70" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
                    </svg>
                    <p class="text-white/90">
                        Drag and drop an image, or
                        <button
                            class="text-white hover:text-white/80 font-medium underline"
                            on:click={() => fileInput?.click()}
                        >
                            browse
                        </button>
                    </p>
                </div>
            {/if}
        </div>
    </div>
    
    <!-- Continue Button -->
    <button 
        class="btn bg-white lg:text-xl text-lg font-bold text-[#f35087] mt-10"
        on:click={handleContinue}
        disabled={!backgroundImage}
    >
        Continue
    </button>

    <!-- Add this modal markup just before the closing main container div -->
    {#if showModal}
        <div class="fixed inset-0 bg-black/50 flex items-center justify-center p-4 z-50">
            <div class="bg-white rounded-lg p-6 max-w-sm w-full text-center">
                <h3 class="text-xl font-bold text-gray-800 mb-4">Invalid Image Format</h3>
                <p class="text-gray-600 mb-6">
                    Please upload an image with either a 1:1 (square) or 3:1 (banner) aspect ratio.
                </p>
                <button 
                    class="btn bg-[#f35087] text-white hover:bg-[#d93d6f]"
                    on:click={() => {
                        showModal = false;
                        backgroundImage = null;
                    }}
                >
                    Try Again
                </button>
            </div>
        </div>
    {/if}
</div>

<!-- Svelte script section -->
<script lang="ts">
    import { goto } from '$app/navigation';
    
    let isDragging = false;
    let fileInput: HTMLInputElement;
    let backgroundImage: string | null = null;
    let showModal = false;
    
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
            processFile(file);
        }
    }
    
    function handleFileSelect(event: Event) {
        const input = event.target as HTMLInputElement;
        const file = input.files?.[0];
        
        if (file && file.type.startsWith('image/')) {
            processFile(file);
        }
    }
    
    function isValidAspectRatio(width: number, height: number): boolean {
        const ratio = width / height;
        const tolerance = 0.1; // Allow 10% deviation from exact ratios
        
        // Check for 1:1 ratio (square)
        if (Math.abs(ratio - 1) <= tolerance) return true;
        
        // Check for 3:1 ratio (banner)
        if (Math.abs(ratio - 3) <= tolerance) return true;
        
        return false;
    }
    
    async function processFile(file: File) {
        const maxWidth = window.innerWidth >= 640 ? 500 : 250;
        const maxHeight = maxWidth;

        const img = new Image();
        const reader = new FileReader();

        reader.onload = () => {
            img.src = reader.result as string;
        };

        img.onload = () => {
            // Check aspect ratio before processing
            if (!isValidAspectRatio(img.width, img.height)) {
                showModal = true;
                return;
            }

            const canvas = document.createElement('canvas');
            let width = img.width;
            let height = img.height;

            // Calculate new dimensions while maintaining aspect ratio
            if (width > height) {
                if (width > maxWidth) {
                    height = (height * maxWidth) / width;
                    width = maxWidth;
                }
            } else {
                if (height > maxHeight) {
                    width = (width * maxHeight) / height;
                    height = maxHeight;
                }
            }

            canvas.width = width;
            canvas.height = height;

            const ctx = canvas.getContext('2d');
            ctx?.drawImage(img, 0, 0, width, height);

            backgroundImage = canvas.toDataURL(file.type);
        };

        reader.readAsDataURL(file);
    }
    
    function handleContinue() {
        if (backgroundImage) {
            goto('/home', {
                state: { backgroundImage }
            });
        }
    }
</script>