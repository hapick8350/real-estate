<script>
  import AcquisitionTaxCalculator from './lib/AcquisitionTaxCalculator.svelte';
  import TransferTaxCalculator from './lib/TransferTaxCalculator.svelte';
  import { onMount } from 'svelte';

  let activeTab = 'acquisition';
  let currentYear = new Date().getFullYear();

  onMount(() => {
    document.title = '부동산 세금 계산기';
  });
</script>

<main>
  <div class="container">
    <header>
      <h1>🏠 부동산 세금 계산기</h1>
      <p class="subtitle">취득세 및 양도소득세를 간편하게 계산해보세요</p>
    </header>

    <div class="tab-container">
      <button 
        class="tab-button {activeTab === 'acquisition' ? 'active' : ''}"
        on:click={() => activeTab = 'acquisition'}
      >
        📥 취득세 계산
      </button>
      <button 
        class="tab-button {activeTab === 'transfer' ? 'active' : ''}"
        on:click={() => activeTab = 'transfer'}
      >
        📤 양도소득세 계산
      </button>
    </div>

    <div class="calculator-container">
      {#if activeTab === 'acquisition'}
        <AcquisitionTaxCalculator />
      {:else}
        <TransferTaxCalculator />
      {/if}
    </div>

    <footer>
      <p>© {currentYear} 부동산 세금 계산기 - 최신 세법 기준으로 계산됩니다</p>
    </footer>
  </div>
</main>

<style>
  .container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  }

  header {
    text-align: center;
    margin-bottom: 40px;
  }

  h1 {
    color: #7f8c8d;
    font-size: 2.5rem;
    margin-bottom: 10px;
    font-weight: 900;
    text-shadow: 0 2px 4px rgba(0, 0, 0, 0.15);
  }

  .subtitle {
    color: #95a5a6;
    font-size: 1.2rem;
    margin: 0;
    font-weight: 700;
    text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
  }

  .tab-container {
    display: flex;
    justify-content: center;
    margin-bottom: 30px;
    gap: 10px;
  }

  .tab-button {
    padding: 15px 30px;
    font-size: 1.1rem;
    border: 2px solid #3498db;
    background: white;
    color: #3498db;
    border-radius: 10px;
    cursor: pointer;
    transition: all 0.3s ease;
    font-weight: 600;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  }

  .tab-button:hover {
    background: #3498db;
    color: white;
    transform: translateY(-2px);
    box-shadow: 0 4px 15px rgba(52, 152, 219, 0.3);
  }

  .tab-button.active {
    background: #3498db;
    color: white;
    box-shadow: 0 4px 15px rgba(52, 152, 219, 0.3);
  }

  .calculator-container {
    background: white;
    border-radius: 15px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
    padding: 30px;
    margin-bottom: 30px;
    border: 1px solid #e0e0e0;
  }

  footer {
    text-align: center;
    color: #34495e;
    font-size: 1rem;
    margin-top: 40px;
    font-weight: 500;
  }

  @media (max-width: 768px) {
    .container {
      padding: 15px;
    }

    h1 {
      font-size: 2rem;
    }

    .subtitle {
      font-size: 1.1rem;
    }

    .tab-container {
      flex-direction: column;
      align-items: center;
    }

    .tab-button {
      width: 100%;
      max-width: 300px;
    }

    .calculator-container {
      padding: 20px;
    }
  }
</style>
