<script>
  import werewolf from '$lib/images/werewolf.svg';
  import blackwolf from '$lib/images/black_wolf.svg';
  import cupid from '$lib/images/cupid.svg';
  import guard from '$lib/images/guard.svg';
  import hunter from '$lib/images/hunter.svg';
  import seer from '$lib/images/seer.svg';
  import villager from '$lib/images/villager.svg';
  import whitewolf from '$lib/images/white_wolf.svg';
  import witch from '$lib/images/witch.svg';
  import talkativewolf from '$lib/images/talkative_wolf.svg';
  import { onMount } from 'svelte';

  let step = 'setup'; // setup, reveal-mode, manual-entry, reveal, gm, voyante
  let mode = 'auto'; // auto or manual

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
  let showRoles = false;
  let showMotLoupBavard = false;
  let loupBavardMot = '';

  const motsBavards = [
    "Wallah", "Frere", "Wesh", "Baguette", "Bipolaire", "Chocolatine", "Sah", "Nuit", "Marine Lepen", "Macron", "Pied" 
  ];

  function validateAndChooseMode() {
    const totalRoles = Object.values(roles).reduce((a, b) => a + b, 0);
    if (totalRoles !== numPlayers) {
      alert("Le nombre total de rôles doit correspondre au nombre de joueurs.");
      return;
    }
    step = 'reveal-mode';
  }

  function assignRolesAutomatically() {
    loupBavardMot = motsBavards[Math.floor(Math.random() * motsBavards.length)];
    let rolePool = [];
    for (const [role, count] of Object.entries(roles)) {
      for (let i = 0; i < count; i++) {
        rolePool.push(role);
      }
    }
    shuffle(rolePool);
    assignments = playerNames.map((name, i) => ({ name, role: rolePool[i], eliminated: false }));
    currentRevealIndex = 0;
    step = 'reveal';
  }

  function prepareManualEntry() {
    assignments = playerNames.map((name) => ({ name, role: '', eliminated: false }));
    step = 'manual-entry';
  }

  function validateManualAssignments() {
    const rolesDistribués = assignments.map(a => a.role).filter(r => r !== '');
    if (rolesDistribués.length !== numPlayers) {
      alert("Tous les rôles doivent être remplis.");
      return;
    }
    const countMap = {};
    for (const a of assignments) {
      countMap[a.role] = (countMap[a.role] || 0) + 1;
    }
    for (const role in countMap) {
      if (!roles[role]) continue;
      if (countMap[role] > roles[role]) {
        alert(`Trop de joueurs ont reçu le rôle ${role}. Maximum: ${roles[role]}`);
        return;
      }
    }
    loupBavardMot = motsBavards[Math.floor(Math.random() * motsBavards.length)];
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

  function goToVoyanteView() {
    step = 'voyante';
  }

  function toggleElimination(index) {
    assignments[index].eliminated = !assignments[index].eliminated;
  }

  function resetGameKeepNames() {
  step = 'setup';
  roles = {
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
  assignments = [];
  currentRevealIndex = 0;
  showModal = false;
  showRoles = false;
  showMotLoupBavard = false;
  loupBavardMot = '';
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

        <button class="primary" on:click={validateAndChooseMode}>➡️ Continuer</button>
      {/if}
    </div>

  {:else if step === 'reveal-mode'}
    <div class="card">
      <h1>🤔 Mode de distribution</h1>
      <p>Souhaitez-vous attribuer les rôles aléatoirement ou manuellement ?</p>
      <button class="primary" on:click={() => { mode = 'auto'; assignRolesAutomatically(); }}>🎲 Aléatoire</button>
      <button class="primary" on:click={() => { mode = 'manual'; prepareManualEntry(); }}>✍️ Manuel</button>
      <button on:click={() => step = 'setup'}>⬅️ Retour</button>
    </div>

  {:else if step === 'manual-entry'}
    <div class="card">
      <h1>✍️ Saisie manuelle des rôles</h1>
      {#each assignments as a, i}
        <label>{a.name}:
          <input type="text" bind:value={assignments[i].role} placeholder="Rôle" />
        </label>
      {/each}
      <button class="primary" on:click={validateManualAssignments}>Valider</button>
      <button on:click={() => step = 'reveal-mode'}>⬅️ Retour</button>
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

          {#if assignments[currentRevealIndex].role === 'Villageois'}
            <img src={villager} alt="Image" />
          {/if}
          {#if assignments[currentRevealIndex].role === 'Loup-garou'}
            <img src={werewolf} alt="Image" />
          {/if}
          {#if assignments[currentRevealIndex].role === 'Loup noir'}
            <img src={blackwolf} alt="Image" />
          {/if}
          {#if assignments[currentRevealIndex].role === 'Loup blanc'}
            <img src={whitewolf} alt="Image" />
          {/if}
          {#if assignments[currentRevealIndex].role === 'Voyante'}
            <img src={seer} alt="Image" />
          {/if}
          {#if assignments[currentRevealIndex].role === 'Sorcière'}
            <img src={witch} alt="Image" />
          {/if}
          {#if assignments[currentRevealIndex].role === 'Chasseur'}
            <img src={hunter} alt="Image" />
          {/if}
          {#if assignments[currentRevealIndex].role === 'Garde'}
            <img src={guard} alt="Image" />
          {/if}
          {#if assignments[currentRevealIndex].role === 'Cupidon'}
            <img src={cupid} alt="Image" />
          {/if}
          
          {#if assignments[currentRevealIndex].role === 'Loup bavard'}
            <img src={talkativewolf} alt="Image" />
            <p class="note">Mot à placer discrètement : <strong>{loupBavardMot}</strong></p>
          {/if}
          <button on:click={() => showModal = false}>Fermer</button>
        </div>
      </div>
    {/if}

  {:else if step === 'gm'}
    <div class="card">
      <h1>🧙‍♂️ Vue Maître du Jeu</h1>
      <h2>📜 Répartition des rôles (secrète)</h2>
      <button on:click={() => showRoles = !showRoles}>
        {showRoles ? '🙈 Cacher les rôles' : '👁️ Voir les rôles'}
      </button>

      {#if showRoles}
        <ul>
          {#each assignments as a, i}
            <li style="opacity: {a.eliminated ? 0.5 : 1}">
              <strong>{a.name}</strong> — {a.role}
              <button on:click={() => toggleElimination(i)}>{a.eliminated ? '⚰️ Réanimer' : '💀 Éliminer'}</button>
            </li>
          {/each}
        </ul>
      {/if}

      {#if assignments.some(a => a.role === 'Loup bavard')}
        <button on:click={() => showMotLoupBavard = !showMotLoupBavard}>
          {showMotLoupBavard ? '🙈 Cacher le mot du Loup bavard' : '👁️ Voir le mot du Loup bavard'}
        </button>
        {#if showMotLoupBavard}
          <p class="note">Mot du Loup bavard à surveiller : <strong>{loupBavardMot}</strong></p>
        {/if}
      {/if}

      <button class="primary" on:click={goToVoyanteView}>🔮 Vue Voyante</button>

      <h2>🔄 Ordre d'un tour</h2>
<ol>
  <li>💘 Cupidon choisit deux amoureux</li>
  <li>❤️ Les amoureux sont révélés silencieusement</li>
  <li>🛡️ Le Garde protège un joueur</li>
  <li>🐺 Les Loups (tous types) choisissent une victime</li>
  <li>🔮 La Voyante découvre le rôle d’un joueur</li>
  <li>🧪 La Sorcière décide d’utiliser ses potions</li>
  <li>🎯 Le Chasseur tire (si tué pendant la nuit)</li>
</ol>

<button on:click={resetGameKeepNames}>🔄 Réinitialiser la partie (garder les noms)</button>

    </div>

  {:else if step === 'voyante'}
    <div class="card">
      <h1>🔮 Vue de la Voyante</h1>
      <p>Cliquez sur une carte pour révéler le rôle du joueur.</p>
      <div class="grid">
        {#each assignments as a, i}
          <div class="card-voyante" on:click={() => a.reveal = !a.reveal} style="opacity: {a.eliminated ? 0.4 : 1}">
            {#if a.reveal}
              <strong>{a.name}</strong><br />
              {#if a.role === 'Villageois'}
            <img src={villager} alt="Image" />
          {/if}
          {#if a.role === 'Loup-garou'}
            <img src={werewolf} alt="Image" />
          {/if}
          {#if a.role === 'Loup noir'}
            <img src={blackwolf} alt="Image" />
          {/if}
          {#if a.role === 'Loup blanc'}
            <img src={whitewolf} alt="Image" />
          {/if}
          {#if a.role === 'Voyante'}
            <img src={seer} alt="Image" />
          {/if}
          {#if a.role === 'Sorcière'}
            <img src={witch} alt="Image" />
          {/if}
          {#if a.role === 'Chasseur'}
            <img src={hunter} alt="Image" />
          {/if}
          {#if a.role === 'Garde'}
            <img src={guard} alt="Image" />
          {/if}
          {#if a.role === 'Cupidon'}
            <img src={cupid} alt="Image" />
          {/if}
          
          {#if a.role === 'Loup bavard'}
            <img src={talkativewolf} alt="Image" />
          {/if}
              {a.role}
            {:else}
              <strong>{a.name}</strong><br />
            {/if}
          </div>
        {/each}
      </div>
      <button on:click={() => step = 'gm'}>⬅️ Retour MJ</button>
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
    max-width: 700px;
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

  .grid {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    justify-content: center;
    margin-top: 1rem;
  }

  .card-voyante {
    background: #444;
    padding: 1rem;
    border-radius: 10px;
    width: 150px;
    text-align: center;
    cursor: pointer;
    box-shadow: 0 5px 10px rgba(0,0,0,0.2);
  }
</style>
