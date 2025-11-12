---
layout: post
title: "Direct Mapped Cache Sim"
author: Siddham
date: 2025-11-10
last_updated: 2025-11-10
categories: [Computer-Science, Programming]
tags: [C++, Computer-Architecture]
# series_title:
# prev_post:
# next_post:
comments: true
---
# Cache Simulator Online Demo

<p>Enter memory addresses (one per line, space-separated, or comma-separated). Supports hex (e.g., 0x140A0) or decimal.</p>

<textarea id="addressInput" rows="10" cols="50" placeholder="e.g., 0x4000 0x4004 0x8000 0x4000 0x8000 0x4000"></textarea>
<br>
<button onclick="runSimulation()">Run Simulation</button>
<br>

<h2 id="outputHeader" style="display:none;">Results</h2>
<pre id="outputArea" style="background-color: #f4f4f4; padding: 10px; border: 1px solid #ddd;"></pre>

<script>
    // 1. Initialize the Wasm Module
    // Ensure the path to the Wasm is correct for your Jekyll setup
    const moduleFactory = window.CacheSimulator({ 
        locateFile: (path, prefix) => {
            if (path.endsWith('.wasm')) {
                // Adjust this path if cache_sim.wasm is in a different location
                return '{{ site.baseurl }}/assets/wasm/cache_sim.wasm'; 
            }
            return prefix + path;
        }
    });

    // 2. Wait for the module to be ready
    moduleFactory().then(instance => {
        window.simulator = instance;
        console.log("WebAssembly Cache Simulator loaded successfully.");
    }).catch(e => {
        console.error("Failed to load Wasm module:", e);
        document.getElementById('outputArea').textContent = "ERROR: Could not load the simulator.";
    });

    // 3. Function to call the exported C++ logic
    function runSimulation() {
        if (!window.simulator) {
            document.getElementById('outputArea').textContent = "Simulator is still loading. Please wait a moment.";
            return;
        }

        const input = document.getElementById('addressInput').value;
        const outputArea = document.getElementById('outputArea');
        const outputHeader = document.getElementById('outputHeader');

        // Clear previous output and show header
        outputArea.textContent = "Running simulation...";
        outputHeader.style.display = 'block';

        try {
            // Call the C++ function exported via Embind
            const results = window.simulator.runSimulation(input);
            outputArea.textContent = results;
        } catch (e) {
            outputArea.textContent = "A critical error occurred during simulation: " + e.message;
        }
    }
</script>

<script src="{{ site.baseurl }}/assets/wasm/cache_sim.js"></script>
