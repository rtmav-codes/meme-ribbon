<!-- Landing/Splash Page -->

<!-- Main container with gradient background -->
<div class="h-screen w-screen bg-[radial-gradient(ellipse_at_center,_var(--tw-gradient-stops))] from-rose-300 via-pink-300 to-pink-100 text-white p-4 overflow-y-auto">
    <!-- Content wrapper -->
    <div class="flex flex-col items-center justify-start max-w-screen-xl mx-auto">
        
        <h1 class="text-3xl font-bold text-center text-[#f35087] mt-5 mb-3"> 🎀 Meme Generator</h1>

        <!-- Image Canvas Container - Added min-height to maintain size -->
        <div class="relative bg-white shadow-lg rounded-lg overflow-hidden flex-shrink-0" 
             style="width: {canvasWidth}px; height: {canvasHeight}px">
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
                class="absolute transition-all duration-200 cursor-move {isDragging ? 'drop-shadow-2xl scale-105' : ''}"
                style="
                    left: {position.x - (ribbonWidth * scale) / 2}px;
                    top: {position.y - (ribbonHeight * scale) / 2}px;
                    transform: rotate({rotation}deg) scale({scale});
                    user-select: none;
                    transition: filter 0.1s ease-out, transform 0.1s ease-out;
                "
                on:mousedown={startDrag}
                on:touchstart|preventDefault={startDrag}
            />
        </div>

        <!-- Controls Panel -->
        <div class="mt-4 bg-white/20 backdrop-blur-sm p-4 rounded-lg w-full max-w-xl"> 
            <!-- Position Controls -->
            <div class="flex flex-col gap-4 mb-4">
                <div>
                    <label for="x-position" class="text-sm">X Position</label>
                    <input 
                        id="x-position"
                        type="range" 
                        min="0" 
                        max={canvasWidth}
                        step="5"
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
                        max={canvasHeight}
                        step="5"
                        bind:value={position.y} 
                        class="w-full"
                    >
                </div>
            </div>

            <!-- Rotation and Scale Controls -->
            <div class="flex flex-col gap-4 mb-4">
                <div>
                    <label for="rotation" class="text-sm">Rotation</label>
                    <input 
                        id="rotation"
                        type="range" 
                        min="0" 
                        max="360" 
                        step="5"
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
                        step="0.05" 
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
                    Export Meme
                </button>
            </div>
        </div>

        <!-- How to Use Section -->
        <div class="w-full max-w-xl mx-auto mt-8 p-4 bg-white/20 backdrop-blur-sm rounded-lg text-[#e2316c]">
            <h2 class="text-xl font-bold mb-3">How to Use</h2>
            <ul class="list-disc list-inside space-y-2">
                <li>Upload a background image using the "Upload Background" button</li>
                <li>Drag the ribbon directly on the canvas or use the sliders to position it</li>
                <li>Adjust the rotation and size using the respective sliders</li>
                <li>When satisfied, click "Export Meme" to download your creation</li>
            </ul>
        </div>
    </div>
</div>

<!-- Script section -->
<script lang="ts">
    import { onMount } from 'svelte';
    import { page } from '$app/stores';
    
    // Add window width tracking
    let windowWidth: number;
    
    // Modified initial canvas dimensions
    let canvasWidth = windowWidth && windowWidth < 768 ? 250 : 500;
    let canvasHeight = windowWidth && windowWidth < 768 ? 250 : 500;
    
    // Update position based on initial canvas size
    let position = { 
        x: canvasWidth / 2, 
        y: canvasHeight / 2 
    };

    // State management
    let rotation = 0;
    let scale = 1;
    let ribbonWidth = 0;
    let ribbonHeight = 0;
    
    // Get background image from navigation state (replacing the original declaration)
    let backgroundImage = ($page.state?.backgroundImage as string) || null;

    // Add these new variables for drag state
    let isDragging = false;
    let dragStartPos = { x: 0, y: 0 };
    let initialPosition = { x: 0, y: 0 };

    // Handle background image upload
    function handleFileUpload(event: Event) {
        const input = event.target as HTMLInputElement;
        if (input.files && input.files[0]) {
            const reader = new FileReader();
            reader.onload = (e) => {
                const img = new Image();
                img.onload = () => {
                    // Calculate new dimensions maintaining aspect ratio
                    const aspectRatio = img.width / img.height;
                    const maxWidth = windowWidth < 768 ? 250 : 500;
                    const maxHeight = windowWidth < 768 ? 250 : 500;
                    
                    if (aspectRatio > 1) {
                        canvasWidth = maxWidth;
                        canvasHeight = maxWidth / aspectRatio;
                    } else {
                        canvasHeight = maxHeight;
                        canvasWidth = maxHeight * aspectRatio;
                    }
                    
                    // Update position to center of new dimensions
                    position = { 
                        x: canvasWidth / 2, 
                        y: canvasHeight / 2 
                    };
                };
                img.src = e.target?.result as string;
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

        canvas.width = canvasWidth;
        canvas.height = canvasHeight;

        const ribbonImg = new Image();
        ribbonImg.src = '/ribbon.png';

        const loadImages = new Promise<void>((resolve) => {
            ribbonImg.onload = () => {
                if (backgroundImage) {
                    const bgImg = new Image();
                    bgImg.onload = () => {
                        // Draw background
                        ctx.drawImage(bgImg, 0, 0, canvasWidth, canvasHeight);
                        
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
                    ctx.fillRect(0, 0, canvasWidth, canvasHeight);
                    
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

    // Add these new functions for drag functionality
    function startDrag(event: MouseEvent | TouchEvent) {
        event.preventDefault();
        isDragging = true;
        
        const clientX = 'touches' in event ? event.touches[0].clientX : event.clientX;
        const clientY = 'touches' in event ? event.touches[0].clientY : event.clientY;
        
        dragStartPos = { x: clientX, y: clientY };
        initialPosition = { ...position };

        // Add event listeners for move and end events
        window.addEventListener('mousemove', handleDrag, { passive: false });
        window.addEventListener('touchmove', handleDrag, { passive: false });
        window.addEventListener('mouseup', stopDrag);
        window.addEventListener('touchend', stopDrag);
    }

    function handleDrag(event: MouseEvent | TouchEvent) {
        if (!isDragging) return;
        
        event.preventDefault();
        
        const clientX = 'touches' in event ? event.touches[0].clientX : event.clientX;
        const clientY = 'touches' in event ? event.touches[0].clientY : event.clientY;
        
        // Calculate the distance moved
        const deltaX = clientX - dragStartPos.x;
        const deltaY = clientY - dragStartPos.y;

        // Update position
        position = {
            x: Math.min(Math.max(initialPosition.x + deltaX, 0), canvasWidth),
            y: Math.min(Math.max(initialPosition.y + deltaY, 0), canvasHeight)
        };
    }

    function stopDrag() {
        isDragging = false;
        
        // Remove event listeners
        window.removeEventListener('mousemove', handleDrag);
        window.removeEventListener('touchmove', handleDrag);
        window.removeEventListener('mouseup', stopDrag);
        window.removeEventListener('touchend', stopDrag);
    }

    onMount(() => {
        // Add window resize listener
        windowWidth = window.innerWidth;
        window.addEventListener('resize', () => {
            windowWidth = window.innerWidth;
            if (!backgroundImage) {
                // Only update canvas size if no image is uploaded
                canvasWidth = windowWidth < 768 ? 250 : 500;
                canvasHeight = windowWidth < 768 ? 250 : 500;
                position = { 
                    x: canvasWidth / 2, 
                    y: canvasHeight / 2 
                };
            }
        });

        // Initialize any necessary setup
        // Initialize ribbon dimensions
        const ribbonImg = new Image();
        ribbonImg.src = '/ribbon.png';
        ribbonImg.onload = () => {
            ribbonWidth = ribbonImg.width;
            ribbonHeight = ribbonImg.height;
        };
    });
</script>