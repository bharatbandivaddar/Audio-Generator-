<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Audio Extractor</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-900 text-slate-100 min-h-screen flex items-center justify-center p-4">

    <div class="max-w-md w-full bg-slate-800 rounded-2xl shadow-xl p-6 border border-slate-700">
        <div class="text-center mb-6">
            <div class="inline-flex items-center justify-center w-12 h-12 rounded-full bg-indigo-600/20 text-indigo-400 mb-3">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19V6l12-3v13M9 19c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zm12-3c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zM9 10l12-3" />
                </svg>
            </div>
            <h1 class="text-2xl font-bold">Audio Extractor</h1>
            <p class="text-slate-400 text-sm mt-1">Paste a video link to download it as an MP3 file</p>
        </div>

        <form id="extractForm" class="space-y-4">
            <div>
                <label for="url" class="block text-xs font-semibold uppercase tracking-wider text-slate-400 mb-2">Video URL</label>
                <input 
                    type="url" 
                    id="url" 
                    required
                    placeholder="https://www.youtube.com/watch?v=..." 
                    class="w-full px-4 py-3 bg-slate-950 border border-slate-700 rounded-lg text-slate-100 placeholder-slate-500 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent text-sm"
                />
            </div>

            <button 
                type="submit" 
                id="submitBtn"
                class="w-full bg-indigo-600 hover:bg-indigo-500 text-white font-medium py-3 rounded-lg transition duration-200 flex items-center justify-center gap-2 cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed">
                <span>Extract & Download MP3</span>
            </button>
        </form>

        <!-- Status & Progress -->
        <div id="statusContainer" class="mt-4 hidden">
            <div id="statusMessage" class="text-sm p-3 rounded-lg text-center"></div>
        </div>
    </div>

    <script>
        const form = document.getElementById('extractForm');
        const submitBtn = document.getElementById('submitBtn');
        const statusContainer = document.getElementById('statusContainer');
        const statusMessage = document.getElementById('statusMessage');

        function setStatus(text, type) {
            statusContainer.classList.remove('hidden');
            statusMessage.textContent = text;
            if (type === 'loading') {
                statusMessage.className = "text-sm p-3 rounded-lg text-center bg-indigo-950 text-indigo-300 border border-indigo-800 animate-pulse";
            } else if (type === 'error') {
                statusMessage.className = "text-sm p-3 rounded-lg text-center bg-red-950 text-red-300 border border-red-800";
            } else if (type === 'success') {
                statusMessage.className = "text-sm p-3 rounded-lg text-center bg-emerald-950 text-emerald-300 border border-emerald-800";
            }
        }

        form.addEventListener('submit', async (e) => {
            e.preventDefault();
            const urlInput = document.getElementById('url').value;

            submitBtn.disabled = true;
            submitBtn.textContent = 'Processing & Downloading...';
            setStatus('Extracting audio and converting to MP3. This may take a few seconds...', 'loading');

            try {
                const response = await fetch('/api/extract', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ url: urlInput })
                });

                if (!response.ok) {
                    const errorData = await response.json().catch(() => ({ detail: 'Failed to process request' }));
                    throw new Error(errorData.detail || 'Download failed');
                }

                // Extract filename from header or fallback
                let filename = 'extracted_audio.mp3';
                const disposition = response.headers.get('content-disposition');
                if (disposition && disposition.indexOf('filename=') !== -1) {
                    const matches = /filename[^;=\n]*=((['"]).*?\2|[^;\n]*)/.exec(disposition);
                    if (matches != null && matches[1]) {
                        filename = matches[1].replace(/['"]/g, '');
                    }
                }

                // Trigger direct file download in browser
                const blob = await response.blob();
                const downloadUrl = window.URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.style.display = 'none';
                a.href = downloadUrl;
                a.download = filename;
                document.body.appendChild(a);
                a.click();
                window.URL.revokeObjectURL(downloadUrl);
                a.remove();

                setStatus('Download completed!', 'success');
            } catch (err) {
                setStatus(err.message || 'An error occurred while extracting audio.', 'error');
            } finally {
                submitBtn.disabled = false;
                submitBtn.textContent = 'Extract & Download MP3';
            }
        });
    </script>
</body>
</html>
