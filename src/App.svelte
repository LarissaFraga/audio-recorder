<script>
  
  let count = 0;
  let recording = false;
  let records = [];
  let mediaRecorder = null;

  navigator.mediaDevices
    .getUserMedia({ audio: true })
    .then((stream) => {
      mediaRecorder = new MediaRecorder(stream);
      let chunks = [];

      mediaRecorder.ondataavailable = function (e) {
        chunks.push(e.data);
      };

      mediaRecorder.onstop = function (e) {
        const blob = new Blob(chunks, { type: "audio/ogg; codecs=opus" });
        chunks = [];
        const audioURL = URL.createObjectURL(blob);
        records = [...records, { id: ++count, audioURL: audioURL }];
      };
    })
    .catch((err) => console.log(err));

  function startRecording() {
    recording = true;
    mediaRecorder.start();
  }

  function stopRecording() {
    recording = false;
    mediaRecorder.stop();
  }

  function deleteRecord(record) {
    records = records.filter((rec) => rec !== record);
  }
</script>

<main>
  <div class="alingCenter">
    <h1>Audio recorder</h1>

    <div class="controls">
      <button
        disabled={mediaRecorder === null || recording}
        on:click={startRecording}>Record</button
      >
      <button disabled={!recording} on:click={stopRecording}>Stop</button>
      {#if recording}
        <span class="recording">Recording</span>
      {:else}
        <span>Waiting</span>
      {/if}
    </div>

    <div class="records">
      {#each records as record}
        <div class="record">
          <span>Record {record.id}</span>
          <audio controls src={record.audioURL}/>
          <button on:click={() => deleteRecord(record)}>Delete</button>
        </div>
      {/each}
    </div>
  </div>
</main>

<style>
  main {
    background-color: white;
    height: 100%;
  }

  .alingCenter {
    text-align: center;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  .record {
    display: flex;
    align-items: center;
    gap: 16px;
    margin-bottom: 16px;
  }

  .controls {
    margin-bottom: 2rem;
  }

  .controls button {
    margin-right: 1rem;
  }

  .recording {
    color: red;
    font-weight: bold;
    animation: blinker 1.5s linear infinite;
  }

  @keyframes blinker {
    0% {
      color: red;
    }
    50% {
      color: black;
    }
    100% {
      color: red;
    }
  }
</style>
