<!-- Landing/Splash Page -->

<!-- Main container with gradient background -->
<div class="h-screen w-screen bg-[radial-gradient(ellipse_at_center,_var(--tw-gradient-stops))] from-rose-300 via-pink-300 to-pink-100 text-white p-4 flex flex-col items-center justify-start">

    <h1 class="text-2xl font-bold">Meme Generator</h1>
    <!-- Image Canvas Container -->
    <div class="relative w-[500px] h-[500px] bg-white shadow-lg rounded-lg overflow-hidden">
        {#if backgroundImage}
            <img 
                src={backgroundImage} 
                alt="Background" 
                class="absolute w-full h-full object-cover"
            />
        {/if}
        
        <img 
            src="/ribbon.png" 
            alt="Ribbon" 
            class="absolute transition-all duration-200"
            style="
                left: {position.x - (ribbonWidth * scale) / 2}px;
                top: {position.y - (ribbonHeight * scale) / 2}px;
                transform: rotate({rotation}deg) scale({scale});
            "
        />
    </div>

    <!-- Controls Panel -->
    <div class="mt-4 bg-white/20 backdrop-blur-sm p-4 rounded-lg">
        <!-- Position Controls -->
        <div class="grid grid-cols-2 gap-4 mb-4">
            <div>
                <label for="x-position" class="text-sm">X Position</label>
                <input 
                    id="x-position"
                    type="range" 
                    min="0" 
                    max={xLimit} 
                    bind:value={position.x} 
                    class="w-full"
                >
            </div>
            <div>
                <label for="y-position" class="text-sm">Y Position</label>
                <input 
                    id="y-position"
                    type="range" 
                    min="0" 
                    max={yLimit} 
                    bind:value={position.y} 
                    class="w-full"
                >
            </div>
        </div>

        <!-- Rotation and Scale Controls -->
        <div class="grid grid-cols-2 gap-4 mb-4">
            <div>
                <label for="rotation" class="text-sm">Rotation</label>
                <input 
                    id="rotation"
                    type="range" 
                    min="0" 
                    max="360" 
                    bind:value={rotation} 
                    class="w-full"
                >
            </div>
            <div>
                <label for="size" class="text-sm">Size</label>
                <input 
                    id="size"
                    type="range" 
                    min="0.1" 
                    max="2" 
                    step="0.1" 
                    bind:value={scale} 
                    class="w-full"
                >
            </div>
        </div>

        <!-- File Upload and Export -->
        <div class="flex gap-4">
            <label class="btn btn-sm">
                Upload Background
                <input 
                    type="file" 
                    accept="image/*" 
                    on:change={handleFileUpload} 
                    class="hidden"
                >
            </label>
            <button class="btn btn-sm" on:click={exportImage}>
                Export PNG
            </button>
        </div>
    </div>
</div>

<!-- Script section -->
<script lang="ts">
    import { onMount } from 'svelte';
    
    // State management
    let position = { x: 250, y: 250 };
    let rotation = 0;
    let scale = 1;
    let backgroundImage: string | null = null;
    let ribbonWidth = 0;
    let ribbonHeight = 0;

    // Handle background image upload
    function handleFileUpload(event: Event) {
        const input = event.target as HTMLInputElement;
        if (input.files && input.files[0]) {
            const reader = new FileReader();
            reader.onload = (e) => {
                backgroundImage = e.target?.result as string;
            };
            reader.readAsDataURL(input.files[0]);
        }
    }

    /**
     * Exports the current canvas state as a PNG image.
     * Handles both background and no-background scenarios while maintaining
     * proper transformation order for the ribbon overlay.
     */
    function exportImage() {
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');
        if (!ctx) return;

        canvas.width = 500;
        canvas.height = 500;

        const ribbonImg = new Image();
        ribbonImg.src = '/ribbon.png';

        const loadImages = new Promise<void>((resolve) => {
            ribbonImg.onload = () => {
                if (backgroundImage) {
                    const bgImg = new Image();
                    bgImg.onload = () => {
                        // Draw background
                        ctx.drawImage(bgImg, 0, 0, 500, 500);
                        
                        // Draw ribbon with correct positioning
                        ctx.save();
                        ctx.translate(position.x, position.y);
                        ctx.rotate((rotation * Math.PI) / 180);
                        ctx.scale(scale, scale);
                        ctx.drawImage(
                            ribbonImg,
                            -ribbonImg.width / 2,
                            -ribbonImg.height / 2
                        );
                        ctx.restore();
                        resolve();
                    };
                    bgImg.src = backgroundImage;
                } else {
                    ctx.fillStyle = 'white';
                    ctx.fillRect(0, 0, 500, 500);
                    
                    ctx.save();
                    ctx.translate(position.x, position.y);
                    ctx.rotate((rotation * Math.PI) / 180);
                    ctx.scale(scale, scale);
                    ctx.drawImage(
                        ribbonImg,
                        -ribbonImg.width / 2,
                        -ribbonImg.height / 2
                    );
                    ctx.restore();
                    resolve();
                }
            };
        });

        loadImages.then(() => {
            const link = document.createElement('a');
            link.download = 'meme.png';
            link.href = canvas.toDataURL('image/png');
            link.click();
        });
    }

    onMount(() => {
        // Initialize any necessary setup
        // Initialize ribbon dimensions
        const ribbonImg = new Image();
        ribbonImg.src = '/ribbon.png';
        ribbonImg.onload = () => {
            ribbonWidth = ribbonImg.width;
            ribbonHeight = ribbonImg.height;
        };
    });

    // Calculate position limits based on ribbon size and scale
    $: xLimit = 1000 - (ribbonWidth * scale);
    $: yLimit = 1000 - (ribbonHeight * scale);
</script>