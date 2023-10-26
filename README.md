# video-conference-website
#html code
<!DOCTYPE html>
<html>
<head>
    <title>Video Conference</title>
    <script src="https://media.twiliocdn.com/sdk/js/video/v2/twilio-video.min.js"></script>
</head>
<body>
    <h1>Video Conference</h1>
    <div>
        <label for="identity">Your Name:</label>
        <input type="text" id="identity">
        <button id="join">Join</button>
    </div>
    <div id="video-container"></div>
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const identityInput = document.getElementById('identity');
            const joinButton = document.getElementById('join');
            const videoContainer = document.getElementById('video-container');

            joinButton.addEventListener('click', () => {
                const identity = identityInput.value;
                fetch('/token', {
                    method: 'POST',
                    body: new URLSearchParams({ identity: identity }),
                    headers: {
                        'Content-Type': 'application/x-www-form-urlencoded',
                    },
                })
                .then(response => response.json())
                .then(data => {
                    Twilio.Video.connect(data.token, {
                        name: 'MyRoom',
                        video: true,
                        audio: true,
                    }).then(room => {
                        room.on('participantConnected', participant => {
                            participant.tracks.forEach(publication => {
                                if (publication.isSubscribed) {
                                    const trackElement = publication.track.attach();
                                    videoContainer.appendChild(trackElement);
                                }
                            });
                        });
                        room.on('participantDisconnected', participant => {
                            participant.tracks.forEach(publication => {
                                if (publication.isSubscribed) {
                                    publication.track.detach().forEach(element => element.remove());
                                }
                            });
                        });
                    });
                });
            });
        });
    </script>
</body>
</html>
