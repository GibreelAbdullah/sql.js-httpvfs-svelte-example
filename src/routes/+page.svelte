<script>
  import { onMount } from "svelte";

  let pageSize = "1024";

  /**
   * @type {((configs: import("sql.js-httpvfs/dist/sqlite.worker").SplitFileConfig[], workerUrl: string, wasmUrl: string, maxBytesToRead?: number) => Promise<import("sql.js-httpvfs").WorkerHttpvfs>) | ((arg0: { from: string; config: { serverMode: string; url: string; requestChunkSize: number; }; }[], arg1: string, arg2: string) => any)}
   */
  let createWorker;
  onMount(async () => {
    const { createDbWorker } = await import("sql.js-httpvfs");
    createWorker = createDbWorker;
  });

  let sqlQuery = `SELECT * from titles WHERE title_id = "tt0055435";`;

  let dbUrl =
    "https://nishad.github.io/sql.js-httpvfs-playground/db/imdb-titles-100000_1024_indexed.db";

  const workerUrl = new URL(
    "sql.js-httpvfs/dist/sqlite.worker.js",
    import.meta.url,
  );
  const wasmUrl = new URL("sql.js-httpvfs/dist/sql-wasm.wasm", import.meta.url);
  let querying = false;

  /**
   * @param {string} sqlQuery
   * @param {import("sql.js-httpvfs").WorkerHttpvfs} worker
   */
  async function queryDb(sqlQuery, worker) {
    const result = await worker.db.query(sqlQuery);
    const bytesRead = await worker.worker.bytesRead;
    const stats = await worker.worker.getStats();
    return { result, bytesRead, stats };
  }

  async function runQueryTest() {
    querying = true;

    const worker = await createWorker(
      [
        {
          from: "inline",
          config: {
            serverMode: "full",
            url: dbUrl,
            requestChunkSize: Number(pageSize),
          },
        },
      ],
      workerUrl.toString(),
      wasmUrl.toString(),
    );

    let queryData = queryDb(sqlQuery, worker);
    await queryData.then((data) => {
      console.log(data);
    });
    querying = false;
  }
</script>

<div class="p-6">
  <button on:click={runQueryTest} class="btn variant-filled">
    {#if querying}
      <div class="placeholder">Running</div>
    {:else}
      Run Test
    {/if}
  </button>
  <div>Check console logs for results</div>
</div>
