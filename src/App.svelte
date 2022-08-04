<script>
  import pluralize from "pluralize";
  import Ribbon from "./Ribbon.svelte";

  const API = "https://api.github.com";
  const p = pluralize;

  let token = "";
  let repositories = [];
  let marked = [];

  async function getUserRepos(page = 1) {
    const response = await fetch(`${API}/user/repos?page=${page}`, {
      headers: {
        Authorization: `token ${token}`,
      },
    });

    return await response.json();
  }

  async function fetchRepositories() {
    repositories = [];

    for (let page = 1; ; page++) {
      const chunk = await getUserRepos(page);

      repositories = [...repositories, ...chunk];

      if (chunk.length === 0) {
        break;
      }

      if (page > 5) {
        break;
      }
    }
  }

  async function deleteRepositories() {
    for (const repo of repositories) {
      if (marked.includes(repo.id) === false) {
        continue;
      }

      await fetch(`${API}/repos/${repo.owner.login}/${repo.name}`, {
        method: "DELETE",
        headers: {
          Authorization: `token ${token}`,
        },
      });
    }

    return fetchRepositories();
  }
</script>

<header>
  <h1>Clean Slate</h1>
  <p>Delete unwanted GitHub repositories. Fast.</p>

  <Ribbon />
</header>

<main>
  <section>
    <header>
      <h2>Step 1. Create a GitHub token</h2>
      <p>
        Generate a new <strong>Personal Access Token</strong> in your
        <a
          ref="noopener noreferrer"
          href="https://github.com/settings/tokens/new?scopes=repo,delete_repo"
          target="_blank"
        >
          GitHub Developer Settings
        </a> to allow this application access and manage your repositories.
      </p>

      <p>
        Make sure you select <samp>repo</samp> permission group and
        <samp>delete_repo</samp> as well.
      </p>
    </header>

    <form>
      <label>
        Enter the token:
        <input type="password" placeholder="ghp_…" bind:value={token} />
      </label>
      <input
        type="button"
        value="Update"
        disabled={!token}
        on:click={fetchRepositories}
      />
    </form>
  </section>

  <section>
    <header>
      <h2>Step 2. Select repositories you want to delete</h2>
      <p>Use repository labels to help you make decision.</p>
    </header>
  </section>

  <section>
    <header>
      <h2>Step 3. Review and confirm</h2>
      <p>
        Double check the selected repositories. Once you press <samp
          >Delete</samp
        > there is no way back.
      </p>
    </header>

    <ul>
      {#each repositories as repository (repository.id)}
        <li>
          <label>
            <input
              type="checkbox"
              bind:group={marked}
              value={repository.id}
              disabled={repository.permissions.admin === false}
            />
            {repository.full_name}

            {#if repository.template}
              <sup>template</sup>
            {/if}

            {#if repository.private}
              <sup>private</sup>
            {/if}

            {#if repository.archived}
              <sup>archived</sup>
            {/if}

            {#if repository.fork}
              <sup>fork</sup>
            {/if}

            {#if repository.open_issues_count}
              <sup>{p("issue", repository.open_issues_count, true)}</sup>
            {/if}

            {#if repository.forks_count}
              <sup>{p("fork", repository.forks_count, true)}</sup>
            {/if}

            {#if repository.stargazers_count}
              <sup>{p("star", repository.stargazers_count, true)}</sup>
            {/if}

            {#if repository.watchers_count}
              <sup>{p("watcher", repository.watchers_count, true)}</sup>
            {/if}
          </label>
        </li>
      {/each}
    </ul>

    <label>
      <input
        type="button"
        value="Delete"
        disabled={!marked.length}
        on:click={deleteRepositories}
      />

      <i>It may take some time for repositories to disappear from this list</i>
    </label>
  </section>
</main>

<style>
  sup {
    text-decoration: underline;
    text-decoration-style: dashed;
  }
</style>
