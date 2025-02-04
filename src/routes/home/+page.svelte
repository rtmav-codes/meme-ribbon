<!-- Landing/Splash Page -->

<!-- Main container - updated to remove fixed height and width -->
<div class="min-h-screen w-full bg-[radial-gradient(ellipse_at_center,_var(--tw-gradient-stops))] from-rose-300 via-pink-300 to-pink-100 text-white p-4">
    <!-- Content wrapper - updated padding -->
    <div class="flex flex-col items-center justify-start max-w-screen-xl mx-auto gap-4">
        
        <h1 class="text-2xl md:text-3xl font-bold text-center text-[#f35087]">🎀 Meme Generator</h1>

        <!-- Image Canvas Container - updated size classes -->
        <div class="relative bg-white shadow-lg rounded-lg overflow-hidden flex-shrink-0 w-[200px] h-[200px] sm:w-[300px] sm:h-[300px] md:w-[500px] md:h-[500px]" 
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
                    left: {position.x}px;
                    top: {position.y}px;
                    transform: translate(-50%, -50%) rotate({rotation}deg) scale({scale});
                    user-select: none;
                    transition: filter 0.1s ease-out, transform 0.1s ease-out;
                "
                on:mousedown={startDrag}
                on:touchstart|preventDefault={startDrag}
            />
        </div>

        <!-- Controls Panel -->
        <div class="bg-white/20 backdrop-blur-sm p-3 md:p-4 rounded-lg w-full max-w-[250px] md:max-w-xl"> 
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
            <div class="flex flex-col gap-4">
                <label class="btn btn-sm">
                    <span class="whitespace-normal text-center">Upload Background</span>
                    <input 
                        type="file" 
                        accept="image/*" 
                        on:change={handleFileUpload} 
                        class="hidden"
                    >
                </label>
                <div class="flex flex-row gap-2 items-center">
                    <button class="btn btn-sm flex-1" on:click={() => exportImage(false)}>
                        <span class="whitespace-normal text-center">Export 1:1</span>
                    </button>
                    <button class="btn btn-sm flex-1" on:click={() => exportImage(true)}>
                        <span class="whitespace-normal text-center">Export 3:1</span>
                    </button>
                </div>
            </div>
        </div>

        <!-- Error Modal -->
        {#if showError}
            <div class="fixed inset-0 bg-black/50 flex items-center justify-center z-50">
                <div class="bg-white p-6 rounded-lg max-w-sm mx-4">
                    <h3 class="text-lg font-bold text-gray-900 mb-2">Invalid Image Ratio</h3>
                    <p class="text-gray-700 mb-4">Please upload an image with either a 1:1 or 3:1 aspect ratio.</p>
                    <button 
                        class="btn btn-sm w-full"
                        on:click={() => showError = false}
                    >
                        Okay
                    </button>
                </div>
            </div>
        {/if}

        <!-- How to Use Section -->
        <div class="w-full max-w-[250px] md:max-w-xl mx-auto p-3 md:p-4 bg-white/20 backdrop-blur-sm rounded-lg text-[#e2316c]">
            <h2 class="text-xl font-bold mb-3">How to Use</h2>
            <ul class="list-disc list-inside space-y-2">
                <li>Upload a background image (1:1 or 3:1 ratio) using the "Upload Background" button</li>
                <li>Drag the ribbon directly on the canvas or use the sliders to position it</li>
                <li>Adjust the rotation and size using the respective sliders</li>
                <li>Choose between 1:1 or 3:1 export ratio and click to download your creation</li>
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
    
    // Update initial canvas dimensions with new sizes
    let canvasWidth = window.innerWidth < 640 ? 200 : 
                      window.innerWidth < 768 ? 300 : 500;
    let canvasHeight = canvasWidth; // Keep it square initially
    
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

    let showError = false;

    // Modified handleFileUpload function
    function handleFileUpload(event: Event) {
        const input = event.target as HTMLInputElement;
        if (input.files && input.files[0]) {
            const reader = new FileReader();
            reader.onload = (e) => {
                const img = new Image();
                img.onload = () => {
                    const aspectRatio = img.width / img.height;
                    
                    // Check for valid aspect ratios (allowing small margin of error)
                    const isValidRatio = Math.abs(aspectRatio - 1) < 0.1 || Math.abs(aspectRatio - 3) < 0.1;
                    
                    if (!isValidRatio) {
                        showError = true;
                        input.value = ''; // Reset input
                        return;
                    }

                    const maxSize = windowWidth < 640 ? 200 : 
                                  windowWidth < 768 ? 300 : 500;
                    
                    canvasWidth = Math.min(maxSize, img.width);
                    canvasHeight = canvasWidth / aspectRatio;
                    
                    position = { 
                        x: canvasWidth / 2, 
                        y: canvasHeight / 2 
                    };
                    
                    backgroundImage = e.target?.result as string;
                };
                img.src = e.target?.result as string;
            };
            reader.readAsDataURL(input.files[0]);
        }
    }

    /**
     * Scales down an image if it exceeds maximum dimensions
     */
    function scaleImage(img: HTMLImageElement, maxWidth: number, maxHeight: number) {
        let width = img.width;
        let height = img.height;
        
        // Calculate aspect ratio
        const aspectRatio = width / height;

        // Scale down if image is too large
        if (width > maxWidth || height > maxHeight) {
            if (aspectRatio > 1) {
                width = maxWidth;
                height = width / aspectRatio;
            } else {
                height = maxHeight;
                width = height * aspectRatio;
            }
        }

        // Create temporary canvas for scaling
        const tempCanvas = document.createElement('canvas');
        tempCanvas.width = width;
        tempCanvas.height = height;
        const tempCtx = tempCanvas.getContext('2d');
        
        if (!tempCtx) return null;
        
        // Use better image smoothing
        tempCtx.imageSmoothingEnabled = true;
        tempCtx.imageSmoothingQuality = 'high';
        
        // Draw scaled image
        tempCtx.drawImage(img, 0, 0, width, height);
        
        return tempCanvas;
    }

    // Modified exportImage function
    function exportImage(isWide: boolean) {
        const container = document.querySelector('.relative') as HTMLElement;
        if (!container) return;
        
        const rect = container.getBoundingClientRect();
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');
        if (!ctx) return;

        const scaleFactor = window.innerWidth <= 768 ? 3 : 2;
        
        // Set canvas dimensions based on export ratio
        if (isWide) {
            canvas.width = rect.width * scaleFactor * 3;
            canvas.height = rect.width * scaleFactor;
        } else {
            canvas.width = rect.width * scaleFactor;
            canvas.height = rect.width * scaleFactor;
        }
        
        // Enable high-quality scaling
        ctx.imageSmoothingEnabled = true;
        ctx.imageSmoothingQuality = 'high';
        
        const ribbonImg = new Image();
        ribbonImg.crossOrigin = 'anonymous';
        ribbonImg.src = '/ribbon.png';

        const loadImages = new Promise<void>((resolve) => {
            ribbonImg.onload = () => {
                if (backgroundImage) {
                    const bgImg = new Image();
                    bgImg.crossOrigin = 'anonymous';
                    bgImg.onload = () => {
                        // Draw background with scaling
                        ctx.drawImage(bgImg, 0, 0, canvas.width, canvas.height);
                        
                        // Draw ribbon with correct positioning and scaling
                        ctx.save();
                        ctx.translate(position.x * scaleFactor, position.y * scaleFactor);
                        ctx.rotate((rotation * Math.PI) / 180);
                        ctx.scale(scale * scaleFactor, scale * scaleFactor);
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
                    // Fill white background
                    ctx.fillStyle = 'white';
                    ctx.fillRect(0, 0, canvas.width, canvas.height);
                    
                    // Draw ribbon with correct positioning and scaling
                    ctx.save();
                    ctx.translate(position.x * scaleFactor, position.y * scaleFactor);
                    ctx.rotate((rotation * Math.PI) / 180);
                    ctx.scale(scale * scaleFactor, scale * scaleFactor);
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
            // Use maximum quality for PNG export
            link.href = canvas.toDataURL('image/png', 1.0);
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
        windowWidth = window.innerWidth;
        window.addEventListener('resize', () => {
            windowWidth = window.innerWidth;
            if (!backgroundImage) {
                const maxSize = windowWidth < 640 ? 200 : 
                               windowWidth < 768 ? 300 : 500;
                canvasWidth = maxSize;
                canvasHeight = maxSize;
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