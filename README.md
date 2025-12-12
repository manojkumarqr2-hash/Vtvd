<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Smart YouTube Downloader</title>
    <style>
        /* --- CSS STYLES --- */
        :root {
            --primary: #ff0000;
            --bg: #121212;
            --card: #1e1e1e;
            --text: #ffffff;
        }
        body {
            font-family: sans-serif;
            background-color: var(--bg);
            color: var(--text);
            display: flex;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            padding-top: 40px;
        }
        .app-container {
            width: 90%;
            max-width: 550px;
            text-align: center;
        }

        /* 1. SEARCH SECTION */
        .search-box {
            background: var(--card);
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.5);
            margin-bottom: 20px;
        }
        input {
            width: 70%;
            padding: 12px;
            border-radius: 6px;
            border: 1px solid #333;
            background: #2b2b2b;
            color: white;
            outline: none;
        }
        .btn-find {
            padding: 12px 20px;
            background: #333;
            color: white;
            border: 1px solid #555;
            border-radius: 6px;
            cursor: pointer;
            font-weight: bold;
        }
        .btn-find:hover { background: #444; }

        /* 2. RESULT SECTION (Hidden Initially) */
        #result-area {
            display: none; /* KEY: This hides everything until searched */
            background: var(--card);
            border-radius: 12px;
            overflow: hidden;
            animation: fadeIn 0.5s;
        }
        
        /* Video Metadata */
        .video-meta {
            padding: 0;
            position: relative;
        }
        .video-meta img {
            width: 100%;
            display: block;
        }
        .meta-content {
            padding: 15px;
            text-align: left;
        }
        .meta-content h3 { margin: 0 0 5px 0; font-size: 16px; }
        .stats { color: #aaa; font-size: 13px; }

        /* 3. DOWNLOAD CONTROLS (Appears After) */
        .download-controls {
            padding: 20px;
            background: #252525;
            border-top: 1px solid #333;
        }
        select {
            padding: 12px;
            background: #111;
            color: white;
            border: 1px solid #444;
            border-radius: 6px;
            margin-bottom: 10px;
            width: 100%;
        }
        .btn-download {
            width: 100%;
            padding: 15px;
            background: var(--primary);
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 18px;
            font-weight: bold;
            cursor: pointer;
            text-transform: uppercase;
        }
        .btn-download:hover { background: #cc0000; }

        /* The Hidden Widget */
        iframe {
            width: 100%;
            height: 400px;
            border: none;
            display: none;
            background: white;
            margin-top: 10px;
        }

        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
    </style>
</head>
<body>

<div class="app-container">
    
    <!-- STEP 1: Search -->
    <div class="search-box">
        <h2 style="margin-top:0; color:#ff0000;">YouTube Downloader</h2>
        <input type="text" id="urlInput" placeholder="Paste Link..." />
        <button class="btn-find" onclick="findVideo()">Find Video</button>
    </div>

    <!-- STEP 2: Result & Download (Hidden by default) -->
    <div id="result-area">
        
        <!-- Metadata -->
        <div class="video-meta">
            <img id="thumb" src="" alt="Cover">
            <div class="meta-content">
                <h3 id="vid-title">Loading Title...</h3>
                <div class="stats">
                    <span id="channel">Channel</span> • 
                    <span id="views">0</span> Views
                </div>
            </div>
        </div>

        <!-- Download Buttons -->
        <div class="download-controls">
            <label style="color:#aaa; font-size:12px;">Choose Format:</label>
            <select id="formatSelect">
                <option value="mp3">Audio (MP3) - Music</option>
                <option value="1080">Video (MP4) - 1080p HD</option>
                <option value="720">Video (MP4) - 720p</option>
                <option value="4k">Video (MP4) - 4K Ultra</option>
            </select>
            
            <button class="btn-download" onclick="startDownload()">⬇ Download Now</button>
            
            <!-- Widget loads here -->
            <iframe id="downloader-frame"></iframe>
        </div>
    </div>

</div>

<script>
    const API_KEY = 'AIzaSyCsTXSU5j9bJdsTnXzHZ8j_Yyp6s0ujXXA';

    function extractID(url) {
        var match = url.match(/^.*((youtu.be\/)|(v\/)|(\/u\/\w\/)|(embed\/)|(watch\?))\??v?=?([^#&?]*).*/);
        return (match && match[7].length == 11) ? match[7] : false;
    }

    function formatNum(x) {
        return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    }

    // STEP 1: Find the Video Details
    async function findVideo() {
        const url = document.getElementById('urlInput').value;
        const resultArea = document.getElementById('result-area');
        
        if (!url) { alert("Please paste a link first"); return; }
        
        const videoId = extractID(url);
        if (!videoId) { alert("Invalid URL"); return; }

        // Fetch Data
        try {
            const apiUrl = `https://www.googleapis.com/youtube/v3/videos?part=snippet,statistics&id=${videoId}&key=${API_KEY}`;
            const res = await fetch(apiUrl);
            const data = await res.json();

            if (data.items && data.items.length > 0) {
                const item = data.items[0];
                
                // Fill Details
                document.getElementById('vid-title').innerText = item.snippet.title;
                document.getElementById('channel').innerText = item.snippet.channelTitle;
                document.getElementById('views').innerText = formatNum(item.statistics.viewCount);
                document.getElementById('thumb').src = item.snippet.thumbnails.high.url;

                // REVEAL THE DOWNLOAD SECTION
                resultArea.style.display = "block";
            } else {
                alert("Video not found (Check API Key or Link)");
            }
        } catch (e) {
            console.log(e);
            // Even if API fails, we show the download box so user can try downloading
            resultArea.style.display = "block";
            document.getElementById('vid-title').innerText = "Video Details Unavailable";
        }
    }

    // STEP 2: Trigger the Download Widget
    function startDownload() {
        const url = document.getElementById('urlInput').value;
        const format = document.getElementById('formatSelect').value;
        const frame = document.getElementById('downloader-frame');

        // Load the widget only when button is clicked
        const widgetUrl = `https://loader.to/api/card/?url=${encodeURIComponent(url)}&f=${format}`;
        
        frame.src = widgetUrl;
        frame.style.display = "block";
    }
</script>

</body>
</html>
