<script>
  const API = "https://api.github.com";

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
</header>

<main>
  <section>
    <header>
      <h2>Step 1. Create a GitHub token</h2>
      <p>
        Generate a new <strong>Personal Access Token</strong> in your
        <a
          ref="noopener noreferrer"
          href="https://github.com/settings/tokens"
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
