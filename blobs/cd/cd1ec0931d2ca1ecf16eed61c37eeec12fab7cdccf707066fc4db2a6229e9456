const DEBUG = false;

if (!DEBUG) {
    console.log = function() {};
    console.warn = function() {};
}

let player = null;
let currentYoutubeId = null;
let targetVolume = 0;
let isPlaying = false;
let preferredQuality = 'small'; // Default to small (240p)

// ============================================
// Weapon Jam Sound System (Web Audio API)
// ============================================
let audioContext = null;
const jamSoundBuffers = {}; // Cache for decoded audio buffers

// Initialize Web Audio API context
function initAudioContext() {
    if (!audioContext) {
        audioContext = new (window.AudioContext || window.webkitAudioContext)();
        console.log("[Poggy WeaponJam] AudioContext initialized, state: " + audioContext.state);
    }
    // Resume if suspended (browser autoplay policy)
    if (audioContext.state === 'suspended') {
        audioContext.resume().then(() => {
            console.log("[Poggy WeaponJam] AudioContext resumed");
        }).catch(e => console.error("[Poggy WeaponJam] Error resuming AudioContext:", e));
    }
    return audioContext;
}

// Preload a jam sound file using Web Audio API
function preloadJamSound(soundFile) {
    if (jamSoundBuffers[soundFile]) return; // Already loaded
    
    const ctx = initAudioContext();
    const url = 'sfx/weaponjam/' + soundFile;
    
    fetch(url)
        .then(response => {
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            return response.arrayBuffer();
        })
        .then(arrayBuffer => ctx.decodeAudioData(arrayBuffer))
        .then(audioBuffer => {
            jamSoundBuffers[soundFile] = audioBuffer;
            console.log("[Poggy WeaponJam] Preloaded sound: " + soundFile);
        })
        .catch(err => {
            console.error("[Poggy WeaponJam] Error preloading sound " + soundFile + ":", err);
        });
}

// Play a weapon jam sound with volume based on distance
function playJamSound(soundFile, volume) {
    console.log("[Poggy WeaponJam] Playing sound: " + soundFile + " at volume: " + volume);
    
    const ctx = initAudioContext();
    
    // Check if we have the buffer cached
    if (jamSoundBuffers[soundFile]) {
        playBufferedSound(jamSoundBuffers[soundFile], volume);
    } else {
        const url = 'sfx/weaponjam/' + soundFile;
        // Load and play on the fly if not preloaded
        fetch(url)
            .then(response => {
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status} (Check file path or restart server)`);
                }
                return response.arrayBuffer();
            })
            .then(arrayBuffer => ctx.decodeAudioData(arrayBuffer))
            .then(audioBuffer => {
                jamSoundBuffers[soundFile] = audioBuffer; // Cache it
                playBufferedSound(audioBuffer, volume);
            })
            .catch(err => {
                console.error("[Poggy WeaponJam] Error loading sound " + soundFile + ":", err);
            });
    }
}

// Play a decoded audio buffer with specified volume
function playBufferedSound(audioBuffer, volume) {
    const ctx = initAudioContext();
    
    // Create buffer source
    const source = ctx.createBufferSource();
    source.buffer = audioBuffer;
    
    // Create gain node for volume control
    const gainNode = ctx.createGain();
    gainNode.gain.value = Math.max(0, Math.min(1, volume));
    
    // Connect: source -> gain -> destination
    source.connect(gainNode);
    gainNode.connect(ctx.destination);
    
    // Play the sound
    source.start(0);
    console.log("[Poggy WeaponJam] Sound playing successfully");
}

// YouTube IFrame API ready callback
function onYouTubeIframeAPIReady() {
    console.log("[Poggy Music] YouTube IFrame API ready triggered.");
    
    player = new YT.Player('player', {
        height: '360',
        width: '640',
        playerVars: {
            'autoplay': 1, // Ensure autoplay is 1
            'controls': 0,
            'disablekb': 1,
            'fs': 0,
            'modestbranding': 1,
            'playsinline': 1,
            'rel': 0,
            'showinfo': 0,
            'origin': window.location.origin
        },
        events: {
            'onReady': onPlayerReady,
            'onStateChange': onPlayerStateChange,
            'onError': onPlayerError
        }
    });
}

function onPlayerReady(event) {
    console.log("[Poggy Music] YouTube player ready event received.");
    
    // Set quality preference
    if (player && player.setPlaybackQuality) {
        player.setPlaybackQuality(preferredQuality);
    }
    
    if (isPlaying && currentYoutubeId) {
        console.log("[Poggy Music] Player ready, attempting to resume/play video: " + currentYoutubeId);
        event.target.playVideo();
        updateVolume();
    }
}

function onPlayerStateChange(event) {
    // -1 (unstarted), 0 (ended), 1 (playing), 2 (paused), 3 (buffering), 5 (video cued).
    const states = {
        '-1': 'Unstarted',
        '0': 'Ended',
        '1': 'Playing',
        '2': 'Paused',
        '3': 'Buffering',
        '5': 'Video Cued'
    };
    console.log("[Poggy Music] Player State Change: " + (states[event.data] || event.data));

    // Enforce quality on play
    if (event.data === YT.PlayerState.PLAYING) {
        if (player && player.setPlaybackQuality) {
            player.setPlaybackQuality(preferredQuality);
        }
    }

    // If video ended and should loop, replay it
    if (event.data === YT.PlayerState.ENDED && isPlaying) {
        console.log("[Poggy Music] Video ended, looping...");
        player.playVideo();
    }
}

function onPlayerError(event) {
    // 2: Invalid param, 5: HTML5 error, 100: Not found, 101/150: Embedded disallowed
    console.error("[Poggy Music] YouTube player error code:", event.data);
    if (event.data === 150 || event.data === 101) {
        console.error("[Poggy Music] This video does not allow playback in embedded players.");
    }
}

// Smooth volume transitions
function updateVolume() {
    if (player && player.setVolume) {
        const vol = Math.round(targetVolume * 100);
        player.setVolume(vol);
    }
}

// Handle messages from client
window.addEventListener('message', function(event) {
    const data = event.data;
    console.log("[Poggy Music] NUI Message received type: " + data.type);
    
    switch(data.type) {
        case 'init':
            console.log("[Poggy Music] System initialized via NUI");
            if (data.quality) {
                preferredQuality = data.quality;
                console.log("[Poggy Music] Preferred quality set to: " + preferredQuality);
            }
            break;
            
        case 'play':
            console.log("[Poggy Music] Play request: ID=" + data.youtubeId + ", Vol=" + data.volume + ", Start=" + (data.startSeconds || 0));
            
            if (player && data.youtubeId !== currentYoutubeId) {
                currentYoutubeId = data.youtubeId;
                isPlaying = true;
                targetVolume = data.volume || 0.5;
                
                console.log("[Poggy Music] Loading new video...");
                player.loadVideoById({
                    videoId: data.youtubeId,
                    startSeconds: data.startSeconds || 0,
                    suggestedQuality: preferredQuality
                });
                
                updateVolume();
            } else if (player && !isPlaying) {
                console.log("[Poggy Music] Resuming existing video...");
                isPlaying = true;
                targetVolume = data.volume || 0.5;
                
                // If a specific start time was requested on resume/restart, seek to it
                if (data.startSeconds !== undefined && data.startSeconds !== null) {
                    player.seekTo(data.startSeconds, true);
                }
                
                player.playVideo();
                updateVolume();
            } else if (!player) {
                console.warn("[Poggy Music] Player object not yet created!");
                // Store intent to play when ready
                currentYoutubeId = data.youtubeId;
                isPlaying = true;
                targetVolume = data.volume || 0.5;
                // Note: startSeconds handling for delayed init would require storing it in a global var, 
                // but usually player is ready by the time zones trigger.
            }
            break;
            
        case 'stop':
            console.log("[Poggy Music] Stopping playback");
            
            if (player) {
                player.stopVideo();
                // Also clear the video to save memory if possible, or just stop
                player.clearVideo(); 
                isPlaying = false;
                currentYoutubeId = null;
                targetVolume = 0;
                updateVolume();
            }
            break;
            
        case 'updateVolume':
            targetVolume = data.volume || 0;
            updateVolume();
            break;
        
        case 'playJamSound':
            // Play weapon jam sound with specified volume
            if (data.sound && data.volume !== undefined) {
                playJamSound(data.sound, data.volume);
            }
            break;
        
        case 'preloadJamSounds':
            // Preload jam sounds on init for faster playback
            if (data.sounds && Array.isArray(data.sounds)) {
                data.sounds.forEach(function(soundFile) {
                    preloadJamSound(soundFile);
                });
            }
            break;
            
        case 'updateAOP':
            const aopDisplay = document.getElementById('aop-display');
            const aopText = document.getElementById('aop-text');
            const aopPlayers = document.getElementById('aop-players');
            
            if (data.visible && data.zoneName) {
                aopText.textContent = '[AREA OF PLAY]: ' + data.zoneName;
                
                // Update player count if provided
                if (data.playerCount !== undefined) {
                    aopPlayers.textContent = '[Players]: ' + data.playerCount;
                }
                
                aopDisplay.classList.add('visible');
                console.log("[Poggy AOP] Displaying zone: " + data.zoneName);
            } else {
                aopDisplay.classList.remove('visible');
                console.log("[Poggy AOP] Hiding AOP display");
            }
            break;

        case 'aopSetSide': {
            // /aop l|r — flip the wrapper to the left or right edge.
            var aopEl = document.getElementById('aop-display');
            if (!aopEl) break;
            if (data.side === 'r') {
                aopEl.classList.add('aop-right');
            } else {
                aopEl.classList.remove('aop-right');
            }
            console.log('[Poggy AOP] Side set to: ' + data.side);
            break;
        }

        case 'updateDutyCount': {
            var dutyLaw = document.getElementById('duty-law');
            var dutyDoc = document.getElementById('duty-doc');
            var dutySep = document.getElementById('duty-sep');
            var dutyRow = document.getElementById('duty-row');
            var lawOn   = (data.lawCount || 0) > 0;
            var docOn   = (data.docCount || 0) > 0;

            if (dutyLaw) dutyLaw.textContent = lawOn ? '★ LAW [' + data.lawCount + ']' : '';
            if (dutyDoc) dutyDoc.textContent = docOn ? '✚ DOC [' + data.docCount + ']' : '';
            if (dutySep) dutySep.textContent = (lawOn && docOn) ? ' | ' : '';
            if (dutyRow) dutyRow.style.display = (lawOn || docOn) ? 'flex' : 'none';
            console.log('[Poggy DutyCount] Law: ' + data.lawCount + ', Doc: ' + data.docCount);
            break;
        }
    }
});

// Smooth volume updates
setInterval(function() {
    if (player && isPlaying) {
        updateVolume();
    }
}, 100);
