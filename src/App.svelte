<script lang="ts">
  import Button from "./Button.svelte";
  let goal = 0;
  let temp = 0;
  let upperMrgn = 0;
  let lowerMrgn = 0;
  const toggleInputDisabled = (id: string) => {
    let element = document.getElementById(id) as HTMLInputElement;
    if (element) {
      if (element.disabled) {
        element.disabled = false;
      } else {
        element.disabled = true;
      }
    }
  };
  const setTemp = () => {
    const newTemp = window.prompt("new temp:");
    if (newTemp && !isNaN(+newTemp)) {
      toggleInputDisabled("setTempBtn");
      // check validity
      fetch(`/api/set_temp`, {
        method: "POST",
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
        body: newTemp,
      })
        .then(() => toggleInputDisabled("setTempBtn"))
        .finally(() => update());
    }
  };
  const setUpperMrgn = () => {
    const newUpperMrgn = window.prompt("new upper margin:");
    if (newUpperMrgn && !isNaN(+newUpperMrgn)) {
      toggleInputDisabled("setUpperMrgnBtn");
      // check validity
      fetch(`/api/set_upper_margin`, {
        method: "POST",
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
        body: newUpperMrgn,
      })
        .then(() => toggleInputDisabled("setUpperMrgnBtn"))
        .finally(() => update());
    }
  };
  const setLowerMrgn = () => {
    const newLowerMrgn = window.prompt("new lower margin:");
    if (newLowerMrgn && !isNaN(+newLowerMrgn)) {
      toggleInputDisabled("setLowerMrgnBtn");
      // check validity
      fetch(`/api/set_lower_margin`, {
        method: "POST",
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
        body: newLowerMrgn,
      })
        .then(() => toggleInputDisabled("setLowerMrgnBtn"))
        .finally(() => update());
    }
  };
  const processResponse = async (body: Promise<any>) => {
    try {
      const rb = await body;
      const reader = rb.getReader();
      const stream = new ReadableStream({
        start(controller) {
          function push() {
            reader.read().then(({ done, value }) => {
              if (done) {
                controller.close();
                return;
              }
              controller.enqueue(value);
              push();
            });
          }
          push();
        },
      });
      const result_3 = await new Response(stream, {
        headers: { "Content-Type": "text/plain" },
      }).text();
      return result_3;
    } catch (err) {
      console.log(`couldn't convert response to text`);
      console.log(err);
      return "0 0 0 0";
    }
  };
  const update = async () => {
    try {
      const updateBody = fetch("/update").then(
        (res) => res.body,
        () => {
          console.log("aaa");
        }
      );

      const updateData = await processResponse(updateBody);
      [temp, goal, lowerMrgn, upperMrgn] = updateData
        .split(" ")
        .map((str) => +str);
    } catch (error) {
      [temp, goal, lowerMrgn, upperMrgn] = [-15, 25, 0.5, 0.5];
    }
  };
  setInterval(() => update(), 3000);
</script>

<main>
  <h1>warm water</h1>
  <p>let's get some temperature regulated</p>
  <div class="outputs max-600">
    <div class="output-block">
      <div class="output-block output-subblock">
        goal:
        <output id="goalOutput" name="goalOutput">{goal ?? 0}°C</output>
      </div>
      <div class="output-block output-subblock">
        current: <output id="tempOutput" name="tempOutput">{temp ?? 0}°C</output
        >
      </div>
    </div>
    <div class="output-block">
      <div class="output-block output-subblock">
        upper: <output id="upperMrgn" name="upperMrgn"
          >{upperMrgn ?? 0}°C (↑)</output
        >
      </div>
      <div class="output-block output-subblock">
        lower: <output id="lowerMrgn" name="lowerMrgn"
          >{lowerMrgn ?? 0}°C (↓)</output
        >
      </div>
    </div>
  </div>
  <div class="buttons max-600">
    <Button tall onClick={setTemp} value="set temp" id="setTempBtn" />
    <Button onClick={setUpperMrgn} value="set upper" id="setUpperMrgnBtn" />
    <Button onClick={setLowerMrgn} value="set lower" id="setLowerMrgnBtn" />
  </div>
</main>

<style>
  main {
    text-align: center;
    /* padding: 1em; */
    /* max-width: 240px; */
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    height: 100vh;
    align-items: center;
    padding-left: 1rem;
    padding-right: 1rem;
  }
  .max-600{
    max-width: 600px;
  }
  .buttons {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    column-gap: 0.5rem;
    width: 100%;
    padding-bottom: 10vh;
    font-weight: 400;
  }
  .outputs {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    width: 90%;
  }
  .output-block {
    display: flex;
    justify-content: space-between;
  }
  .output-subblock {
    width: 45%;
    align-items: center;
  }
  h1 {
    color: #ff3e00;
    font-size: 5em;
    font-weight: 700;
    margin: 0.5rem;
    line-height: 4rem;
    max-width: 240px;
  }
  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
