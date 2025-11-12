---
layout: post
title: "Direct Mapped Cache Sim"
author: Siddham
date: 2025-11-10
last_updated: 2025-11-10
categories: [Computer-Science, Programming]
tags: [C++, Computer-Architecture]
comments: true
---

# Cache Simulator Online Demo

Enter memory addresses (one per line, space-separated, or comma-separated). Supports hex (e.g., `0x140A0`) or decimal.

<textarea id="addressInput" rows="10" cols="50" placeholder="e.g., 0x4000 0x4004 0x8000 0x4000 0x8000 0x4000"></textarea>
<br>
<button onclick="runSimulation()">Run Simulation</button>
<button onclick="resetSimulation()">Reset Cache</button>
<br>

<h2 id="outputHeader" style="display:none;">Results</h2>
<pre id="outputArea"></pre>

<script src="{{ site.baseurl }}/assets/wasm/cache_sim.js"></script>

<script>
    let simulator = null;
    let runSim = null;
    let resetCache = null;

    // Initialize the WebAssembly module
    createCacheModule({
        locateFile: (path, prefix) => {
            if (path.endsWith('.wasm')) {
                return '{{ site.baseurl }}/assets/wasm/cache_sim.wasm';
            }
            return prefix + path;
        }
    }).then(instance => {
        simulator = instance;
        runSim = instance.cwrap('runSimulation', 'string', ['string']);
        resetCache = instance.cwrap('resetCache', null, []);
        console.log("✓ Cache Simulator loaded successfully");
    }).catch(e => {
        console.error("✗ Failed to load Wasm module:", e);
        document.getElementById('outputArea').textContent = "ERROR: Could not load the simulator. Check console for details.";
    });

    function runSimulation() {
        if (!simulator) {
            document.getElementById('outputArea').textContent = "⏳ Simulator is still loading. Please wait a moment.";
            return;
        }

        const input = document.getElementById('addressInput').value.trim();
        const outputArea = document.getElementById('outputArea');
        const outputHeader = document.getElementById('outputHeader');

        if (!input) {
            outputArea.textContent = "Please enter at least one address.";
            outputHeader.style.display = 'block';
            return;
        }

        try {
            // Call the C++ function via cwrap
            const results = runSim(input);
            outputArea.textContent = results;
            outputHeader.style.display = 'block';
        } catch (e) {
            console.error("Simulation error:", e);
            outputArea.textContent = "❌ A critical error occurred during simulation:\n" + e.message;
            outputHeader.style.display = 'block';
        }
    }

    function resetSimulation() {
        if (simulator) {
            resetCache();
            document.getElementById('outputArea').textContent = "✓ Cache has been reset to initial state.";
            document.getElementById('outputHeader').style.display = 'block';
        }
    }

    // Allow Enter key in textarea
    document.getElementById('addressInput').addEventListener('keydown', function(e) {
        if (e.key === 'Enter' && e.ctrlKey) {
            runSimulation();
        }
    });
</script>
