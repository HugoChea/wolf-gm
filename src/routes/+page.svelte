// Structure de base SvelteKit - fichier App.svelte (ou page principale)

<script>
  import { onMount } from 'svelte';

  let step = 'setup'; // setup, reveal, gm

  let numPlayers = 0;
  let playerNames = [];
  let roles = {
    "Villageois": 0,
    "Loup-garou": 0,
    "Loup noir": 0,
    "Loup bavard": 0,
    "Loup blanc": 0,
    "Voyante": 0,
    "Sorcière": 0,
    "Chasseur": 0,
    "Garde": 0,
    "Cupidon": 0
  };

  let assignments = [];
  let currentRevealIndex = 0;
  let showModal = false;
  let showRolesGM = false;

  function validateAndAssignRoles() {
    const totalRoles = Object.values(roles).reduce((a, b) => a + b, 0);
    if (totalRoles !== numPlayers) {
      alert("Le nombre total de rôles doit correspondre au nombre de joueurs.");
      return;
    }

    let rolePool = [];
    for (const [role, count] of Object.entries(roles)) {
      for (let i = 0; i < count; i++) {
        rolePool.push(role);
      }
    }

    shuffle(rolePool);
    assignments = playerNames.map((name, i) => ({ name, role: rolePool[i] }));
    currentRevealIndex = 0;
    step = 'reveal';
  }

  function shuffle(array) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
  }

  function nextReveal() {
    showModal = false;
    if (currentRevealIndex < assignments.length - 1) {
      currentRevealIndex++;
    } else {
      step = 'gm';
    }
  }
</script>

<main>
  {#if step === 'setup'}
    <div class="card">
      <h1>🎲 Configuration de la partie</h1>
      <label>Nombre de joueurs:
        <input type="number" bind:value={numPlayers} min="1" />
      </label>

      {#if numPlayers > 0}
        <h2>🙋‍♂️ Noms des joueurs</h2>
        {#each Array(numPlayers) as _, i}
          <input type="text" bind:value={playerNames[i]} placeholder="Joueur {i + 1}" />
        {/each}

        <h2>🎭 Sélection des rôles</h2>
        <div class="roles">
          {#each Object.entries(roles) as [role, count]}
            <label>{role}:
              <input type="number" bind:value={roles[role]} min="0" />
            </label>
          {/each}
        </div>

        <button class="primary" on:click={validateAndAssignRoles}>🎬 Lancer la distribution</button>
      {/if}
    </div>

  {:else if step === 'reveal'}
    <div class="card">
      <h1>🃏 Distribution des rôles</h1>
      <p><strong>Joueur :</strong> {assignments[currentRevealIndex].name}</p>
      <button class="reveal" on:click={() => showModal = true}>👁️ Voir mon rôle</button>
      <button on:click={nextReveal}>➡️ Passer au suivant</button>
    </div>

    {#if showModal}
      <div class="modal-overlay" on:click={() => showModal = false}>
        <div class="modal-card" on:click|stopPropagation>
          <h2>Votre rôle</h2>
          <p>{assignments[currentRevealIndex].role}</p>
          <button on:click={() => showModal = false}>Fermer</button>
        </div>
      </div>
    {/if}

  {:else if step === 'gm'}
    <div class="card">
      <h1>🧙‍♂️ Vue Maître du Jeu</h1>
      <h2>📜 Répartition des rôles (secrète)</h2>
      <button on:click={() => showRolesGM = !showRolesGM}>
        {showRolesGM ? '🙈 Cacher les rôles' : '👁️ Afficher les rôles'}
      </button>
      {#if showRolesGM}
        <ul>
          {#each assignments as a}
            <li><strong>{a.name}</strong> — {a.role}</li>
          {/each}
        </ul>
      {/if}
      <p class="note">Ici, on ajoutera l'assistant MJ : gestion des nuits, votes, morts, etc.</p>
    </div>
  {/if}
</main>

<style>
  main {
    font-family: 'Segoe UI', sans-serif;
    padding: 2rem;
    background: linear-gradient(to bottom, #1f1f1f, #3a3a3a);
    min-height: 100vh;
    color: white;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .card {
    background: #2a2a2a;
    border-radius: 1rem;
    padding: 2rem;
    width: 100%;
    max-width: 600px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.5);
  }

  h1, h2 {
    color: #ffdd57;
    margin-bottom: 1rem;
  }

  input {
    display: block;
    margin: 5px 0 15px 0;
    padding: 0.5rem;
    border-radius: 5px;
    border: none;
    width: 100%;
    box-sizing: border-box;
  }

  .roles label {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  button {
    padding: 0.75rem 1.5rem;
    margin: 0.5rem 0;
    border-radius: 8px;
    border: none;
    cursor: pointer;
    background: #444;
    color: white;
    font-size: 1rem;
  }

  button.primary {
    background-color: #4caf50;
  }

  button.reveal {
    background-color: #2196f3;
  }

  ul {
    list-style: none;
    padding-left: 0;
  }

  li {
    margin-bottom: 0.5rem;
  }

  .note {
    margin-top: 1rem;
    font-style: italic;
    color: #ccc;
  }

  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background-color: rgba(0, 0, 0, 0.75);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
  }

  .modal-card {
    background-color: #333;
    padding: 2rem;
    border-radius: 1rem;
    text-align: center;
    color: white;
    box-shadow: 0 10px 20px rgba(0,0,0,0.3);
  }
</style>
