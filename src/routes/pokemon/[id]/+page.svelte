<script lang="ts">
  import { onMount } from "svelte";
  import { goto } from "$app/navigation";
  import type { PageData } from "./$types";
  import { theme } from "$lib/stores/themeStore";
  import TypeBadge from "$lib/components/TypeBadge.svelte";
  import Footer from "$lib/components/Footer.svelte";
  import { fetchAbilityDescription } from "$lib/api/pokemon";

  export let data: PageData;

  const pokemon = data.pokemon;
  let isLoading = true;

  let showMovesModal = false;
  let showAbilityModal = false;
  let fullMovesList: any[] = [];
  let currentAbility: { name: string; description: string } | null = null;

  onMount(() => {
    isLoading = false;
  });

  const goBack = () => {
    goto("/");
  };

  const navigatePokemon = (direction: "next" | "prev") => {
    const newID = direction === "next" ? pokemon.id + 1 : pokemon.id - 1;
    goto(`/pokemon/${newID}`);
  };

  const handleAbilityClick = async (abilityName: string) => {
    const ability = await fetchAbilityDescription(abilityName);
    if (ability) {
      currentAbility = ability;
      showAbilityModal = true;
    }
  };

  const openMovesModal = () => {
    fullMovesList = pokemon.moves;
    showMovesModal = true;
  };

  const formatStatName = (name: string): string => {
    const statNames: Record<string, string> = {
      hp: "HP",
      attack: "Attack",
      defense: "Defense",
      "special-attack": "Sp. Attack",
      "special-defense": "Sp. Defense",
      speed: "Speed",
    };

    return statNames[name] || name;
  };

  const getStatColor = (value: number): string => {
    if (value < 50) return "#ff5959";
    if (value < 80) return "#ffdd57";
    return "#78c850";
  };

  const calculateStatPercent = (value: number): number => {
    return Math.min(Math.max((value / 255) * 100, 5), 100);
  };

  function toggleTheme() {
    theme.update((current) => (current === "light" ? "dark" : "light"));
  }
</script>

<div class="pokemon-detail-container">
  {#if isLoading}
    <div class="loading-container">
      <div class="spinner"></div>
      <p>Loading Pokémon data...</p>
    </div>
  {:else}
    <div class="detail-header">
      <div class="header-top">
        <button class="back-button" on:click={goBack}>
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="20"
            height="20"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <path d="M19 12H5M12 19l-7-7 7-7" />
          </svg>
          Back
        </button>

        <button
          class="theme-toggle"
          on:click={toggleTheme}
          aria-label="Toggle theme"
        >
          {#if $theme === "light"}
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="24"
              height="24"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>
            </svg>
          {:else}
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="24"
              height="24"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <circle cx="12" cy="12" r="5"></circle>
              <line x1="12" y1="1" x2="12" y2="3"></line>
              <line x1="12" y1="21" x2="12" y2="23"></line>
              <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line>
              <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line>
              <line x1="1" y1="12" x2="3" y2="12"></line>
              <line x1="21" y1="12" x2="23" y2="12"></line>
              <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line>
              <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>
            </svg>
          {/if}
        </button>
      </div>

      <h1>
        <span class="pokemon-name">{pokemon.name}</span>
        <span class="pokemon-id">#{pokemon.id.toString().padStart(3, "0")}</span
        >
      </h1>

      <div class="navigation-buttons">
        {#if pokemon.id > 1}
          <button
            class="nav-button prev"
            on:click={() => navigatePokemon("prev")}
          >
            ← Previous
          </button>
        {/if}
        <button
          class="nav-button next"
          on:click={() => navigatePokemon("next")}
        >
          Next →
        </button>
      </div>
    </div>

    <div class="pokemon-content">
      <div class="image-container">
        <div class="image-wrapper">
          <img
            src={pokemon.sprites.other?.["official-artwork"]?.front_default ||
              pokemon.sprites.front_default}
            alt={pokemon.name}
            class="pokemon-image"
          />
        </div>

        <div class="types-container">
          {#each pokemon.types as { type }}
            <TypeBadge type={type.name} />
          {/each}
        </div>

        <div class="physical-info">
          <div class="info-item">
            <span class="info-label">Height</span>
            <span class="info-value">{(pokemon.height / 10).toFixed(1)} m</span>
          </div>
          <div class="info-item">
            <span class="info-label">Weight</span>
            <span class="info-value">{(pokemon.weight / 10).toFixed(1)} kg</span
            >
          </div>
        </div>
      </div>

      <div class="details-card">
        <section class="card-section">
          <h2>Base Stats</h2>
          <div class="stats-container">
            {#each pokemon.stats as stat}
              <div class="stat-item">
                <div class="stat-name">{formatStatName(stat.stat.name)}</div>
                <div class="stat-value">{stat.base_stat}</div>
                <div class="stat-bar">
                  <div
                    class="stat-fill"
                    style="width: {calculateStatPercent(
                      stat.base_stat
                    )}%; background-color: {getStatColor(stat.base_stat)}"
                  ></div>
                </div>
              </div>
            {/each}
          </div>
        </section>

        <section class="card-section">
          <h2>Abilities</h2>
          <div class="abilities-container">
            {#each pokemon.abilities as ability}
              <button
                class="ability-badge"
                on:click={() => handleAbilityClick(ability.ability.name)}
              >
                {ability.ability.name.replace("-", " ")}
                {#if ability.is_hidden}
                  <span class="hidden-ability">(Hidden)</span>
                {/if}
              </button>
            {/each}
          </div>
        </section>

        <section class="card-section">
          <h2>Moves</h2>
          <div class="moves-container">
            {#each pokemon.moves.slice(0, 6) as move}
              <div class="move-badge">
                {move.move.name.replace("-", " ")}
              </div>
            {/each}
            {#if pokemon.moves.length > 6}
              <button class="more-moves" on:click={openMovesModal}>
                +{pokemon.moves.length - 6} more
              </button>
            {/if}
          </div>
        </section>
      </div>
    </div>

    {#if showAbilityModal && currentAbility}
      <div
        class="modal"
        role="dialog"
        tabindex="0"
        on:click|self={() => (showAbilityModal = false)}
        on:keydown={(event) =>
          event.key === "Escape" && (showAbilityModal = false)}
      >
        <div class="modal-content">
          <h3>{currentAbility.name}</h3>
          <p>{currentAbility.description}</p>
          <button on:click={() => (showAbilityModal = false)}>Close</button>
        </div>
      </div>
    {/if}

    {#if showMovesModal}
      <div
        class="modal"
        role="dialog"
        tabindex="0"
        on:click|self={() => (showMovesModal = false)}
        on:keydown={(event) =>
          event.key === "Escape" && (showMovesModal = false)}
      >
        <div class="modal-content">
          <h3>All Moves</h3>
          <div class="moves-grid">
            {#each fullMovesList as move}
              <div class="move-badge">
                {move.move.name.replace("-", " ")}
              </div>
            {/each}
          </div>
          <button on:click={() => (showMovesModal = false)}>Close</button>
        </div>
      </div>
    {/if}
  {/if}
</div>

<Footer />

<style>
  .header-top {
    display: flex;
    justify-content: space-between;
    align-items: center;
    position: relative;
  }

  .pokemon-detail-container {
    max-width: 1000px;
    margin: 0 auto;
    padding: 20px;
    font-family: "Arial", sans-serif;
  }

  .loading-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 60vh;
  }

  @keyframes spin {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
  .spinner {
    width: 50px;
    height: 50px;
    border: 5px solid #f3f3f3;
    border-top: 5px solid #3d7dca;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    margin-bottom: 15px;
    border-color: var(--spinner-border);
    border-top-color: var(--primary-color);
  }

  .detail-header {
    margin-bottom: 30px;
  }

  .back-button {
    background: #ff5350;
    color: white;
    border: none;
    padding: 8px 16px;
    border-radius: 8px;
    font-size: 16px;
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 8px;
    transition:
      transform 0.2s ease,
      background-color 0.2s ease;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  }

  .back-button:hover {
    background: #e74c3c;
    transform: scale(1.05);
  }

  h1 {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 15px;
    margin: 25px 0;
  }

  .pokemon-name {
    font-size: 2.5rem;
    text-transform: capitalize;
    color: var(--text-color);
  }

  .pokemon-id {
    font-size: 1.5rem;
    color: var(--text-color);
    font-weight: normal;
  }

  .pokemon-content {
    display: grid;
    grid-template-columns: 1fr 1.5fr;
    gap: 30px;
  }

  @media (max-width: 768px) {
    .pokemon-content {
      grid-template-columns: 1fr;
    }
  }

  .image-container {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .image-wrapper {
    background: radial-gradient(
      circle,
      rgba(240, 240, 240, 1) 0%,
      rgba(220, 220, 220, 1) 100%
    );
    border-radius: 50%;
    padding: 20px;
    width: 250px;
    height: 250px;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-bottom: 20px;
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
  }

  .pokemon-image {
    max-width: 100%;
    max-height: 100%;
    object-fit: contain;
    filter: drop-shadow(2px 4px 6px rgba(0, 0, 0, 0.2));
  }

  .types-container {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
  }

  .physical-info {
    display: flex;
    gap: 20px;
    margin-top: 15px;
  }

  .info-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    background-color: #f5f5f5;
    padding: 10px 20px;
    border-radius: 10px;
  }

  .info-label {
    font-size: 14px;
    color: #666;
  }

  .info-value {
    font-size: 18px;
    font-weight: bold;
    color: #333;
  }

  .details-card {
    border-radius: 15px;
    overflow: hidden;
    background-color: var(--card-bg);
    box-shadow: 0 4px 8px var(--card-shadow);
  }

  .card-section {
    padding: 20px;
    border-bottom: 1px solid #eee;
  }

  .card-section:last-child {
    border-bottom: none;
  }

  h2 {
    margin: 0 0 15px 0;
    font-size: 1.3rem;
    color: var(--primary-color);
    position: relative;
    padding-bottom: 8px;
  }

  h2::after {
    content: "";
    position: absolute;
    bottom: 0;
    left: 0;
    width: 50px;
    height: 3px;
    background-color: #ffcb05;
    border-radius: 3px;
  }

  .stats-container {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  .stat-item {
    display: grid;
    grid-template-columns: 100px 40px 1fr;
    align-items: center;
    gap: 10px;
  }

  .stat-name {
    font-weight: bold;
    color: var(--text-color);
  }

  .stat-value {
    font-weight: bold;
    text-align: right;
    color: var(--text-color);
  }

  .stat-bar {
    height: 8px;
    background-color: #eee;
    border-radius: 4px;
    overflow: hidden;
  }

  .stat-fill {
    height: 100%;
    border-radius: 4px;
    transition: width 0.5s ease-out;
  }

  .abilities-container,
  .moves-container {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
  }

  .move-badge {
    background-color: #f5f5f5;
    padding: 6px 12px;
    border-radius: 20px;
    font-size: 14px;
    text-transform: capitalize;
    color: #333;
  }

  .hidden-ability {
    font-size: 12px;
    color: #666;
    font-style: italic;
    margin-left: 4px;
    opacity: 0.8;
  }

  .ability-badge,
  .more-moves {
    background-color: #f5f5f5;
    padding: 6px 12px;
    border-radius: 20px;
    font-size: 14px;
    border: none;
    text-transform: capitalize;
    color: #333;
    cursor: pointer;
    transition:
      transform 0.2s ease,
      background-color 0.2s ease;
  }

  .ability-badge:hover,
  .more-moves:hover {
    background-color: #e0e0e0;
    transform: translateY(-2px);
    box-shadow: 0 3px 6px rgba(0, 0, 0, 0.2);
  }

  .navigation-buttons {
    display: flex;
    justify-content: space-between;
    margin-top: 20px;
    gap: 15px;
  }

  .nav-button {
    flex: 1;
    padding: 10px 15px;
    border: none;
    border-radius: 8px;
    background-color: var(--primary-color, #3d7dca);
    color: white;
    font-size: 16px;
    cursor: pointer;
    transition:
      background-color 0.2s ease,
      transform 0.2s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
  }

  .nav-button:hover {
    background-color: var(--primary-color-dark, #2c5f99);
    transform: scale(1.02);
  }

  @keyframes fadeIn {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }
  .modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.6);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
    padding: 20px;
    animation: fadeIn 0.3s ease-out;
  }

  @keyframes scaleIn {
    from {
      transform: scale(0.95);
    }
    to {
      transform: scale(1);
    }
  }
  .modal-content {
    background-color: var(--card-bg, white);
    border-radius: 20px;
    max-width: 800px;
    width: 90%;
    max-height: 80vh;
    overflow-y: auto;
    padding: 25px;
    box-shadow: 0 6px 12px var(--card-shadow, rgba(0, 0, 0, 0.15));
    animation: scaleIn 0.3s ease-out;
  }

  .modal-content h3 {
    margin: 0 0 15px 0;
    font-size: 1.3rem;
    color: var(--primary-color);
    position: relative;
    padding-bottom: 8px;
    text-align: center;
    border-bottom: 2px solid var(--primary-color);
  }

  .moves-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 12px;
    max-height: 60vh;
    overflow-y: auto;
    padding: 10px;
  }

  .moves-grid .move-badge {
    background-color: #f5f5f5;
    padding: 10px 15px;
    border-radius: 25px;
    text-align: center;
    font-size: 16px;
    font-weight: 500;
    text-transform: capitalize;
    transition:
      background-color 0.2s ease,
      transform 0.2s ease;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  }

  .modal-content button {
    display: block;
    width: 100%;
    padding: 12px;
    margin-top: 20px;
    background-color: var(--primary-color, #3d7dca);
    color: white;
    border: none;
    border-radius: 10px;
    font-size: 18px;
    cursor: pointer;
    transition:
      background-color 0.2s ease,
      transform 0.2s ease;
  }

  .modal-content button:hover {
    background-color: var(--primary-color-dark, #2c5f99);
    transform: scale(1.02);
  }
</style>
