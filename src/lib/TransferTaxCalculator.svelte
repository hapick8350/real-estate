<script>
  // 양도소득세 계산을 위한 상태 변수들
  let acquisitionPrice = 0; // 취득가액
  let transferPrice = 0; // 양도가액
  let acquisitionDate = new Date().toISOString().split('T')[0]; // 취득일
  let transferDate = new Date().toISOString().split('T')[0]; // 양도일
  let propertyType = 'apartment'; // 부동산 유형
  let isFirstHome = false; // 첫 주택 여부
  let isLongTerm = false; // 장기보유 여부
  let hasOtherProperties = false; // 다른 부동산 보유 여부

  // 계산 결과
  let capitalGain = 0; // 양도차익
  let deduction = 0; // 공제액
  let taxableIncome = 0; // 과세표준
  let transferTax = 0; // 양도소득세
  let localTax = 0; // 지방소득세
  let totalTax = 0; // 총 세금

  // 부동산 유형 옵션
  const propertyTypes = [
    { value: 'apartment', label: '아파트' },
    { value: 'house', label: '단독주택' },
    { value: 'land', label: '토지' },
    { value: 'commercial', label: '상가/사무실' }
  ];

  // 양도소득세 세율 (2024년 기준)
  const taxRates = [
    { bracket: 12000000, rate: 0.06 }, // 1,200만원 이하: 6%
    { bracket: 46000000, rate: 0.15 }, // 4,600만원 이하: 15%
    { bracket: 88000000, rate: 0.24 }, // 8,800만원 이하: 24%
    { bracket: 150000000, rate: 0.35 }, // 1억5천만원 이하: 35%
    { bracket: 300000000, rate: 0.38 }, // 3억원 이하: 38%
    { bracket: 500000000, rate: 0.40 }, // 5억원 이하: 40%
    { bracket: Infinity, rate: 0.42 }   // 5억원 초과: 42%
  ];

  // 양도소득세 계산 함수
  function calculateTransferTax() {
    if (acquisitionPrice <= 0 || transferPrice <= 0) {
      capitalGain = 0;
      deduction = 0;
      taxableIncome = 0;
      transferTax = 0;
      localTax = 0;
      totalTax = 0;
      return;
    }

    // 양도차익 계산
    capitalGain = transferPrice - acquisitionPrice;

    if (capitalGain <= 0) {
      deduction = 0;
      taxableIncome = 0;
      transferTax = 0;
      localTax = 0;
      totalTax = 0;
      return;
    }

    // 공제액 계산
    deduction = calculateDeduction();

    // 과세표준 계산
    taxableIncome = Math.max(0, capitalGain - deduction);

    // 양도소득세 계산
    transferTax = calculateProgressiveTax(taxableIncome);

    // 지방소득세 계산 (양도소득세의 10%)
    localTax = Math.round(transferTax * 0.1);

    // 총 세금 계산
    totalTax = transferTax + localTax;
  }

  // 공제액 계산 함수
  function calculateDeduction() {
    let deduction = 0;

    // 기본 공제 (600만원)
    deduction += 6000000;

    // 첫 주택 공제 (1,200만원)
    if (isFirstHome && (propertyType === 'apartment' || propertyType === 'house')) {
      deduction += 12000000;
    }

    // 장기보유 공제
    if (isLongTerm) {
      const holdingPeriod = calculateHoldingPeriod();
      if (holdingPeriod >= 3) {
        deduction += Math.min(capitalGain * 0.06, 12000000); // 최대 1,200만원
      }
    }

    return deduction;
  }

  // 보유기간 계산 함수
  function calculateHoldingPeriod() {
    const acquisition = new Date(acquisitionDate);
    const transfer = new Date(transferDate);
    const diffTime = Math.abs(transfer - acquisition);
    const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
    return diffDays / 365; // 년 단위로 반환
  }

  // 누진세 계산 함수
  function calculateProgressiveTax(income) {
    let remainingIncome = income;
    let totalTax = 0;
    let previousBracket = 0;

    for (let i = 0; i < taxRates.length; i++) {
      const currentBracket = taxRates[i].bracket;
      const rate = taxRates[i].rate;

      if (remainingIncome <= 0) break;

      const taxableInThisBracket = Math.min(
        remainingIncome,
        currentBracket - previousBracket
      );

      totalTax += taxableInThisBracket * rate;
      remainingIncome -= taxableInThisBracket;
      previousBracket = currentBracket;
    }

    return Math.round(totalTax);
  }

  // 입력값 변경 시 자동 계산
  $: if (acquisitionPrice > 0 && transferPrice > 0) {
    calculateTransferTax();
  }
</script>

<div class="calculator">
  <h2>📤 양도소득세 계산기</h2>
  
  <div class="input-section">
    <div class="input-group">
      <label for="acquisitionPrice">취득가액 (원)</label>
      <input 
        type="number" 
        id="acquisitionPrice"
        bind:value={acquisitionPrice}
        placeholder="예: 300000000"
        min="0"
      />
    </div>

    <div class="input-group">
      <label for="transferPrice">양도가액 (원)</label>
      <input 
        type="number" 
        id="transferPrice"
        bind:value={transferPrice}
        placeholder="예: 500000000"
        min="0"
      />
    </div>

    <div class="input-group">
      <label for="acquisitionDate">취득일</label>
      <input 
        type="date" 
        id="acquisitionDate"
        bind:value={acquisitionDate}
      />
    </div>

    <div class="input-group">
      <label for="transferDate">양도일</label>
      <input 
        type="date" 
        id="transferDate"
        bind:value={transferDate}
      />
    </div>

    <div class="input-group">
      <label for="propertyType">부동산 유형</label>
      <select id="propertyType" bind:value={propertyType}>
        {#each propertyTypes as type}
          <option value={type.value}>{type.label}</option>
        {/each}
      </select>
    </div>

    <div class="checkbox-group">
      <label class="checkbox-label">
        <input 
          type="checkbox" 
          bind:checked={isFirstHome}
        />
        첫 주택 (1,200만원 추가 공제)
      </label>
    </div>

    <div class="checkbox-group">
      <label class="checkbox-label">
        <input 
          type="checkbox" 
          bind:checked={isLongTerm}
        />
        장기보유 (3년 이상, 6% 추가 공제)
      </label>
    </div>

    <div class="checkbox-group">
      <label class="checkbox-label">
        <input 
          type="checkbox" 
          bind:checked={hasOtherProperties}
        />
        다른 부동산 보유
      </label>
    </div>
  </div>

  {#if acquisitionPrice > 0 && transferPrice > 0}
    <div class="result-section">
      <h3>📊 계산 결과</h3>
      
      <div class="result-grid">
        <div class="result-item">
          <span class="result-label">양도차익</span>
          <span class="result-value {capitalGain > 0 ? 'positive' : 'negative'}">
            {capitalGain.toLocaleString()}원
          </span>
        </div>
        
        <div class="result-item">
          <span class="result-label">공제액</span>
          <span class="result-value">{deduction.toLocaleString()}원</span>
        </div>
        
        <div class="result-item">
          <span class="result-label">과세표준</span>
          <span class="result-value">{taxableIncome.toLocaleString()}원</span>
        </div>
        
        <div class="result-item">
          <span class="result-label">양도소득세</span>
          <span class="result-value">{transferTax.toLocaleString()}원</span>
        </div>
        
        <div class="result-item">
          <span class="result-label">지방소득세</span>
          <span class="result-value">{localTax.toLocaleString()}원</span>
        </div>
        
        <div class="result-item total">
          <span class="result-label">총 세금</span>
          <span class="result-value">{totalTax.toLocaleString()}원</span>
        </div>
      </div>

      <div class="tax-info">
        <h4>💡 세율 정보</h4>
        <div class="tax-brackets">
          {#each taxRates as rate, index}
            <p>
              {#if index === 0}
                {rate.bracket.toLocaleString()}원 이하: {rate.rate * 100}%
              {:else if rate.bracket === Infinity}
                {taxRates[index - 1].bracket.toLocaleString()}원 초과: {rate.rate * 100}%
              {:else}
                {taxRates[index - 1].bracket.toLocaleString()}원 초과 ~ {rate.bracket.toLocaleString()}원: {rate.rate * 100}%
              {/if}
            </p>
          {/each}
        </div>

        <h4>📋 공제 정보</h4>
        <p>• 기본 공제: 600만원</p>
        {#if isFirstHome}
          <p class="deduction">• 첫 주택 공제: 1,200만원</p>
        {/if}
        {#if isLongTerm}
          <p class="deduction">• 장기보유 공제: 양도차익의 6% (최대 1,200만원)</p>
        {/if}
        <p>• 지방소득세: 양도소득세의 10%</p>

        {#if capitalGain > 0}
          <div class="holding-period">
            <h4>📅 보유기간</h4>
            <p>총 보유기간: {calculateHoldingPeriod().toFixed(1)}년</p>
          </div>
        {/if}
      </div>
    </div>
  {/if}
</div>

<style>
  .calculator {
    max-width: 800px;
    margin: 0 auto;
  }

  h2 {
    color: #1a252f;
    text-align: center;
    margin-bottom: 30px;
    font-size: 1.8rem;
    font-weight: 700;
  }

  .input-section {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 20px;
    margin-bottom: 30px;
  }

  .input-group {
    display: flex;
    flex-direction: column;
  }

  .input-group label {
    font-weight: 600;
    color: #1a252f;
    margin-bottom: 8px;
    font-size: 1rem;
  }

  .input-group input,
  .input-group select {
    padding: 12px;
    border: 2px solid #e0e0e0;
    border-radius: 8px;
    font-size: 1rem;
    transition: border-color 0.3s ease;
    background: white;
    color: #2c3e50;
  }

  .input-group input:focus,
  .input-group select:focus {
    outline: none;
    border-color: #e74c3c;
    box-shadow: 0 0 0 3px rgba(231, 76, 60, 0.1);
  }

  .checkbox-group {
    display: flex;
    align-items: center;
    margin-top: 10px;
  }

  .checkbox-label {
    display: flex;
    align-items: center;
    cursor: pointer;
    font-weight: 500;
    color: #1a252f;
    font-size: 1rem;
  }

  .checkbox-label input[type="checkbox"] {
    margin-right: 10px;
    transform: scale(1.2);
  }

  .result-section {
    background: #f8f9fa;
    border-radius: 12px;
    padding: 25px;
    margin-top: 30px;
    border: 1px solid #e0e0e0;
  }

  .result-section h3 {
    color: #1a252f;
    margin-bottom: 20px;
    text-align: center;
    font-size: 1.4rem;
    font-weight: 700;
  }

  .result-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 15px;
    margin-bottom: 25px;
  }

  .result-item {
    background: white;
    padding: 15px;
    border-radius: 8px;
    text-align: center;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    border: 1px solid #e0e0e0;
  }

  .result-item.total {
    background: #e74c3c;
    color: white;
    font-weight: bold;
  }

  .result-label {
    display: block;
    font-size: 0.95rem;
    margin-bottom: 5px;
    opacity: 0.9;
    font-weight: 500;
  }

  .result-value {
    display: block;
    font-size: 1.3rem;
    font-weight: 700;
  }

  .result-value.positive {
    color: #e74c3c;
  }

  .result-value.negative {
    color: #27ae60;
  }

  .tax-info {
    background: white;
    padding: 20px;
    border-radius: 8px;
    border-left: 4px solid #e74c3c;
    border: 1px solid #e0e0e0;
  }

  .tax-info h4 {
    color: #1a252f;
    margin-bottom: 15px;
    font-size: 1.1rem;
    font-weight: 700;
  }

  .tax-info p {
    margin: 8px 0;
    color: #2c3e50;
    font-size: 1rem;
    font-weight: 500;
  }

  .deduction {
    color: #27ae60 !important;
    font-weight: 700;
  }

  .tax-brackets {
    background: #f8f9fa;
    padding: 15px;
    border-radius: 6px;
    margin: 10px 0;
    border: 1px solid #e0e0e0;
  }

  .holding-period {
    margin-top: 20px;
    padding-top: 20px;
    border-top: 1px solid #e0e0e0;
  }

  @media (max-width: 768px) {
    .input-section {
      grid-template-columns: 1fr;
    }

    .result-grid {
      grid-template-columns: 1fr;
    }

    .calculator {
      padding: 0 10px;
    }
  }
</style>
