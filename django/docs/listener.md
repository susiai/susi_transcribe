disable_toc: true

<h1>Translation Listener</h1>
<div id="header"></div>
  <input type="text" id="tenant_id" placeholder="Tenant ID" value="0000">
  <button id="newsessionBtn">New Session</button>
</div>
<div id="transcript-container"></div>

<script>
  let transcriptContainer = document.getElementById('transcript-container');
  let pollingInProgress = false;  // This flag will ensure serialized requests

  function adjustTranscriptContainerHeight() {
    const headerHeight = document.getElementById('header').offsetHeight;
    const windowHeight = window.innerHeight;
    transcriptContainer.style.height = (windowHeight - headerHeight - 80) + 'px';
  }

  document.getElementById('newsessionBtn').addEventListener('click', newSession);
  
  function newSession() {
    while (transcriptContainer.firstChild) {
      transcriptContainer.removeChild(transcriptContainer.firstChild);
    }
  }

  function getLatestTranscript() {
    if (pollingInProgress) {
      return;  // Skip if a request is already in progress
    }

    pollingInProgress = true;  // Set flag indicating polling has started

    const tenant_id = document.getElementById('tenant_id').value;
    let get_latest_transcript_url = `/api/get_latest_transcript?tenant_id=${tenant_id}`;

    fetch(get_latest_transcript_url)
      .then(response => response.json())
      .then(data => {
        // data is a dictionary with keys: chunk_id and objects with attributes: transcript, translated
        // iterate over the keys to get the data
        // console.log(data);

        chunk_ids = Object.keys(data);
        for (i = 0; i < chunk_ids.length; i++) {
          chunk_id = chunk_ids[i];
          transcript_event = data[chunk_id]
          transcript = transcript_event.transcript;
          
          // find the div with the chunk_id
          div = document.getElementById(chunk_id);
          if (div === null) {
            // New chunk ID, add a new line to the transcript container
            const newLine = document.createElement('div');
            newLine.id = chunk_id;
            newLine.textContent = transcript;
            transcriptContainer.appendChild(newLine);
          } else {
            // Same chunk ID, update the existing transcript
            div.textContent = transcript;
          }
        }

        // Scroll to the bottom of the transcript container
        transcriptContainer.scrollTop = transcriptContainer.scrollHeight;
      })
      .catch(error => console.error('Error fetching latest transcript:', error))
      .finally(() => {
        pollingInProgress = false;  // Reset the flag once the polling is done

        // Schedule the next poll after this one is done
        setTimeout(getLatestTranscript, 1000);  // Poll every 1 second
      });
  }

  // Adjust transcript container height on load and resize
  window.addEventListener('load', () => {
    adjustTranscriptContainerHeight();
    getLatestTranscript();  // Start polling when the page loads
  });
  
  window.addEventListener('resize', adjustTranscriptContainerHeight);
</script>
