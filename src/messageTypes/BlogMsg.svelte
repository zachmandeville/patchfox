<script>
  import { navigate, routeLocation } from "../utils.js";

  export let msg;

  let content = msg.value.content;

  let summary = ssb.markdown(content.summary);
  let thumbnail = content.thumbnail || false;
  let title = content.title || false;
  let showBlogpost = false;
  let loading = false;
  let toast = false;
  let toastMsg = "";
  let post = summary;

  let liked = false;

  ssb.votes(msg.key).then(ms => {
    ms.forEach(m => {
      let author = m.value.author;
      if ((author === ssb.feed && m.value.content.vote.value === 1)) {
        liked = true;
      }
    });
  });

  const likeChanged = ev => {
    let v = ev.target.checked;
    if (v) {
      ssb
        .like(msg.key)
        .then(() => console.log("liked", msg.key))
        .catch(() => (liked = false));
    } else {
      ssb
        .unlike(msg.key)
        .then(() => console.log("unliked", msg.key))
        .catch(() => (liked = true));
    }
  };

  const displayBlogPost = ev => {
    loading = true;
    console.log("loading blogpost", content.blog);

    ssb
      .getBlob(content.blog)
      .then(data => {
        post = ssb.markdown(data);
        showBlogpost = true;
      })
      .catch(err => {
        console.error("can't load blog post", err);
        toast = true;
        toastMsg = err;
      });
  };

  const reply = ev => {
    let rootId = msg.value.content.root || msg.key;
    let channel = msg.value.content.channel;
    navigate("/compose", { root: rootId, branch: msg.key, channel });
  };

  const goRoot = ev => {
    let rootId = msg.value.content.root || msg.key;
    navigate("/thread", { thread: rootId });
  };

  const goBranch = ev => {
    let branchId = msg.value.content.branch || msg.key;
    navigate("/thread", { thread: branchId });
  };

  if ($routeLocation == "/thread") {
    setTimeout(displayBlogPost, 100);
  }
</script>

<style>
  div img.is-image-from-blob {
    max-width: 90%;
  }
</style>

{#if thumbnail}
  <div class="card-image">
    <img
      src="http://localhost:8989/blobs/get/{encodeURIComponent(thumbnail)}"
      class="img-responsive"
      alt={title} />
  </div>
{/if}
<div class="card-body">
  {#if title}
    <h1 class="card-title h5">{title}</h1>
  {/if}

  {#if toast}
    <div class="toast toast-error">Can't load blogpost: {toastMsg}</div>
  {/if}
  {#if showBlogpost}
    {@html post}
  {:else}
    {@html summary}
  {/if}
</div>
<div class="card-footer">
  <div class="columns col-gapless">
    <div class="column col-6">
      <label class="form-switch d-inline">
        <input type="checkbox" on:change={likeChanged} checked={liked} />
        <i class="form-icon" />
        Like
      </label>
      {#if msg.value.content.root}
        <span>
          <a
            href="?thread={encodeURIComponent(msg.value.content.root)}#/thread"
            on:click|preventDefault={goRoot}>
            (root)
          </a>
        </span>
      {/if}
      {#if msg.value.content.branch}
        <span>
          <a
            href="?thread={encodeURIComponent(msg.value.content.branch)}#/thread"
            on:click|preventDefault={goBranch}>
            (in reply to)
          </a>
        </span>
      {/if}
    </div>
    <div class="column col-6 text-right">
      <button class="btn" on:click={reply}>Reply</button>
      {#if !showBlogpost}
        <button
          class="btn btn-primary"
          class:locating={loading}
          on:click={displayBlogPost}>
          Read Blogpost
        </button>
      {:else}
        <button
          class="btn btn-primary"
          class:locating={loading}
          on:click={() => (showBlogpost = false)}>
          Close Blogpost
        </button>
      {/if}
    </div>
  </div>

</div>
