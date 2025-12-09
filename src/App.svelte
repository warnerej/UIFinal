<script>
  import { onMount } from 'svelte';
  import World from './lib/world.svelte'
  let mapinstance;
  let query = "";
  let selectedCountry = "United States";
  let selectedCountryNeed = null;
  let customAmount = "";
  let showCardModal = false;
  let cardAmount = null;
  let topCountries = [];

  function updateSelectedCountry() {
    selectedCountry = mapinstance.getSelectedCountry();
    selectedCountryNeed = mapinstance.getCountryNeed(selectedCountry);
  }

  function handleSearch() {
    const foundCountry = mapinstance.highlightCountry(query);
    if (foundCountry) {
      selectedCountry = foundCountry;
      selectedCountryNeed = mapinstance.getCountryNeed(foundCountry);
      // Inform world component to visually mark the selected country
      if (mapinstance && mapinstance.setSelectedCountry) {
        mapinstance.setSelectedCountry(foundCountry);
      }
    }
  }

  function donateCustom(amount) {
    const val = amount ?? customAmount;
    const num = Number(val);
    if (!num || num <= 0) {
      alert('Please enter a valid donation amount');
      return;
    }
    openCardModal(num);
  }

  function openCardModal(amount) {
    cardAmount = amount;
    showCardModal = true;
  }

  function closeCardModal() {
    showCardModal = false;
    cardAmount = null;
  }

  function confirmInsertCard() {
    // Mock processing — replace with real payment integration as needed
    alert(`Card inserted — processing donation of $${cardAmount} to ${selectedCountry}`);
    closeCardModal();
    customAmount = "";
  }

  function handleModalKeydown(e) {
    if (e.key === 'Escape') closeCardModal();
  }

  function getNeedColor(need) {
    if (!need) return "#FF7575";
    const intensity = Math.min(need / 100, 1);
    const r = Math.round(255 * intensity);
    const g = Math.round(117 * (1 - intensity * 0.7));
    const b = Math.round(117 * (1 - intensity * 0.7));
    return `rgb(${r}, ${g}, ${b})`;
  }

  function getNeedLabel(need) {
    if (!need) return "Unknown";
    if (need >= 85) return "Critical";
    if (need >= 70) return "High";
    if (need >= 50) return "Moderate";
    if (need >= 25) return "Low";
    return "Minimal";
  }

  onMount(() => {
    // initialize the world selection color for the default selected country
    if (mapinstance && mapinstance.setSelectedCountry) {
      mapinstance.setSelectedCountry(selectedCountry);
      selectedCountryNeed = mapinstance.getCountryNeed(selectedCountry);
      if (mapinstance.getTopCountries) {
        topCountries = mapinstance.getTopCountries(3);
      }
    }
  });

  function selectTopCountry(country) {
    if (!country) return;
    if (mapinstance && mapinstance.setSelectedCountry) {
      mapinstance.setSelectedCountry(country);
    }
    selectedCountry = country;
    selectedCountryNeed = mapinstance.getCountryNeed(country);
  }

</script>

<svelte:window on:keydown={(e) => showCardModal && handleModalKeydown(e)} />

<main>
<div class="parent">
  <div class="header">
    <div class="search-bar">
      <input
      class="search-input"
      placeholder="Search for a country..."
      bind:value={query}
      on:input={handleSearch}>
    </div>
    <div class="brand">
      <div class="brand-title"> HungryHelper</div>
    </div>
  </div>
  <div class="side-bar">
    {#if selectedCountry}
      <div class="country-info">
        <h2>{selectedCountry}</h2>
        <p>Click on a country to see more information</p>
        {#if selectedCountryNeed}
          <div class="need-indicator">
            <div class="need-bar-container">
              <div class="need-bar" style="width: {selectedCountryNeed}%; background-color: {getNeedColor(selectedCountryNeed)};"></div>
            </div>
            <p class="need-text">
              Food Aid Need: <span class="need-value" style="color: {getNeedColor(selectedCountryNeed)}; font-weight: bold;">{selectedCountryNeed}/100</span>
            </p>
            <p class="need-label">{getNeedLabel(selectedCountryNeed)}</p>
          </div>
        {/if}
      </div>
      <div class="donation-section">
        <h3>Donate to {selectedCountry}</h3>
        <div class="donation-options">
          <button class="donation-btn" on:click={() => openCardModal(5)}>
            <span class="amount">$5</span>
            <span class="label">Donation</span>
          </button>
          <button class="donation-btn" on:click={() => openCardModal(25)}>
            <span class="amount">$25</span>
            <span class="label">Donation</span>
          </button>
          <button class="donation-btn" on:click={() => openCardModal(50)}>
            <span class="amount">$50</span>
            <span class="label">Donation</span>
          </button>
          <button class="donation-btn" on:click={() => openCardModal(100)}>
            <span class="amount">$100</span>
            <span class="label">Donation</span>
          </button>
        </div>
        <input
          type="number"
          placeholder="Custom amount"
          min="1"
          class="custom-donation-input"
          bind:value={customAmount}
          on:keydown={(e) => e.key === 'Enter' && donateCustom()}
        >
        <button class="custom-donation-btn" on:click={() => donateCustom()}>
          Donate Custom Amount
        </button>
      </div>
      <div class="quick-links">
        <h4>Top Need — Quick Select</h4>
        <div class="quick-buttons">
          {#each topCountries as c}
            <button class="quick-btn" on:click={() => selectTopCountry(c)}>
              <div class="q-name">{c}</div>
              <div class="q-need">{mapinstance ? mapinstance.getCountryNeed(c) : ''}/100</div>
            </button>
          {/each}
        </div>
      </div>
    {:else}
      <div class="country-info">
        <p>Click on a country to display information</p>
      </div>
    {/if}
  </div>
  <div class="map">
    <World bind:this={mapinstance} on:countrySelected={updateSelectedCountry}/>
  </div>
  {#if showCardModal}
    <div class="modal-overlay" on:click={closeCardModal}>
      <div class="modal" on:click|stopPropagation>
        <h3>Please insert your card</h3>
        {#if cardAmount}
          <p>Amount: <strong>${cardAmount}</strong> to {selectedCountry}</p>
        {/if}
        <div class="modal-actions">
          <button class="modal-btn primary" on:click={confirmInsertCard}>Insert Card</button>
          <button class="modal-btn" on:click={closeCardModal}>Cancel</button>
        </div>
      </div>
    </div>
  {/if}
</div>


</main>

<style>
.parent {
display: grid;
grid-template-columns: .75fr repeat(3, 1fr);
grid-template-rows: .5fr repeat(4, 1fr);
grid-column-gap: 0px;
grid-row-gap: 0px;
}

.header { grid-area: 1 / 2 / 2 / 5; }
.side-bar { grid-area: 1 / 1 / 6 / 2; }
.map { grid-area: 2 / 2 / 6 / 5; }


.search-bar {
  width: 50%;
  height: 25%;
  margin-left: 1rem;
}
.search-input {
  padding: 0.5em 1em;
  border: 1px solid #aaa;
  border-radius: 0.6em;
  font-size: 1em;
  outline: none;
  background-color: white;
  box-shadow: 0px 2px 4px rgba(0,0,0,0.15);
  transition: box-shadow 150ms, border 150ms;
  width: 100%;
  height: 100%;
}
.search-input:focus {
  border-color: #1e90ff;
  box-shadow: 0px 3px 6px rgba(30,144,255,0.4);
}

.header{
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 100%;
  padding: 0 1.25rem;
  background: var(--header-background);
}

.side-bar{
  background-color: #f5f7fa; /* subtle neutral background to lift cards */
  border-radius: 1em;
  margin-top: 1em;
  height: 90vh;
  padding: 1.25em;
  overflow-y: auto;
}
.header{
  height:15vh
}
.map{
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: var(--map-background);
  border-radius: 1em;
}

.brand { display:flex; align-items:center; gap:0.6rem }
.brand-icon { flex: 0 0 auto }
.brand-title { color: #ffffff; font-weight: 600; font-size: 4rem; font-style: italic; font-family: 'Brush Script MT', 'Lucida Handwriting', 'Pacifico', cursive; }

.country-info {
  color: var(--text-color);
  padding: 1em;
  border-radius: 0.6em;
  background-color: #ffffff;
  margin-top: 1em;
  margin-bottom: 0.75rem;
  border: 1px solid #e6e9ee;
  box-shadow: 0 8px 28px rgba(2,6,23,0.06);
  transition: transform 150ms ease, box-shadow 150ms ease;
}
.country-info:hover { transform: translateY(-3px); box-shadow: 0 14px 40px rgba(2,6,23,0.08); }

.country-info h2 {
  margin: 0 0 0.5em 0;
  font-size: 1.5em;
}
.country-info p {
  margin: 0;
  font-size: 0.95em;
}

.need-indicator {
  margin-top: 1.5em;
  padding: 1em;
  background-color: rgba(0, 0, 0, 0.05);
  border-radius: 0.5em;
}

.need-bar-container {
  width: 100%;
  height: 24px;
  background-color: #e0e0e0;
  border-radius: 0.4em;
  overflow: hidden;
  margin-bottom: 0.8em;
}

.need-bar {
  height: 100%;
  transition: width 300ms ease, background-color 300ms ease;
  border-radius: 0.4em;
}

.need-text {
  margin: 0 0 0.5em 0;
  font-size: 0.95em;
}

.need-value {
  font-size: 1.1em;
}

.need-label {
  margin: 0;
  font-size: 0.9em;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

/* Donation styles - modern, low-color palette */
.donation-section {
  background-color: #ffffff;
  padding: 1rem;
  border-radius: 0.6rem;
  margin-top: 1rem;
  margin-bottom: 0.75rem;
  border: 1px solid #e6e9ee;
  box-shadow: 0 8px 28px rgba(2,6,23,0.06);
  transition: transform 150ms ease, box-shadow 150ms ease;
}
.donation-section:hover { transform: translateY(-3px); box-shadow: 0 14px 40px rgba(2,6,23,0.08); }

.donation-section h3 {
  margin: 0 0 0.75rem 0;
  font-size: 1.05rem;
  color: #111827;
}

.donation-options {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0.6rem;
  margin-bottom: 0.75rem;
}

.donation-btn {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 0.65rem 0.75rem;
  background: #fff;
  color: #111827;
  border: 1px solid #e6e9ee;
  border-radius: 0.5rem;
  font-weight: 600;
  cursor: pointer;
  transition: transform 150ms ease, box-shadow 150ms ease, border-color 150ms ease;
}

.donation-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 18px rgba(16,24,40,0.08);
  border-color: #cdd5df;
}

.donation-btn:active {
  transform: translateY(0);
}

.donation-btn .amount {
  font-size: 1.05rem;
  margin-bottom: 0.2rem;
}

.donation-btn .label {
  font-size: 0.8rem;
  color: #6b7280;
  font-weight: 500;
}

.custom-donation-input {
  width: 100%;
  padding: 0.6rem 0.75rem;
  border: 1px solid #e6e9ee;
  border-radius: 0.5rem;
  font-size: 1rem;
  margin-bottom: 0.6rem;
  box-sizing: border-box;
}

.custom-donation-input:focus {
  outline: none;
  box-shadow: 0 0 0 4px rgba(17,24,39,0.03);
  border-color: #cdd5df;
}

.custom-donation-btn {
  width: 100%;
  padding: 0.7rem;
  background: #111827; /* neutral dark */
  color: #ffffff;
  border: none;
  border-radius: 0.5rem;
  font-weight: 600;
  cursor: pointer;
  transition: transform 150ms ease, box-shadow 150ms ease, opacity 150ms ease;
}

.custom-donation-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(17,24,39,0.08);
}

.custom-donation-btn:active {
  transform: translateY(0);
}

/* Modal styles */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
}
.modal {
  background: #fff;
  padding: 1.25rem;
  border-radius: 0.6rem;
  width: 90%;
  max-width: 420px;
  box-shadow: 0 10px 40px rgba(2,6,23,0.3);
  color: #111827;
}
.modal h3 {
  margin: 0 0 0.5rem 0;
}
.modal p { margin: 0 0 1rem 0; }
.modal-actions { display:flex; gap:0.5rem; justify-content:flex-end; }
.modal-btn { padding:0.55rem 0.8rem; border-radius:0.45rem; border:1px solid #e6e9ee; background:#fff; cursor:pointer; font-weight:600 }
.modal-btn.primary { background:#111827; color:#fff; border:none }

.quick-links { margin-top: 1rem; background:#fff; padding:0.75rem; border-radius:0.6rem; border:1px solid #e6e9ee; box-shadow: 0 8px 24px rgba(2,6,23,0.05); }
.quick-links h4 { margin:0 0 0.5rem 0; font-size:0.95rem }
.quick-buttons { display:flex; flex-direction:column; gap:0.5rem }
.quick-btn { display:flex; justify-content:space-between; align-items:center; padding:0.5rem; border-radius:0.45rem; border:1px solid #e6e9ee; background:#fff; cursor:pointer; font-weight:600 }
.quick-btn:hover { transform: translateY(-2px); box-shadow: 0 6px 18px rgba(16,24,40,0.06) }
.q-name { font-size:0.95rem }
.q-need { font-size:0.85rem; color:#6b7280 }


</style>