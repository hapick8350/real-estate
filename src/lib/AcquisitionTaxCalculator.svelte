<script>
  // 취득세 계산을 위한 상태 변수들
  let propertyValue = 0; // 부동산 가액
  let propertyType = 'apartment'; // 부동산 유형
  let isFirstHome = false; // 첫 주택 여부
  let isNewConstruction = false; // 신축 여부
  let location = 'seoul'; // 지역
  let acquisitionDate = new Date().toISOString().split('T')[0]; // 취득일

  // 계산 결과
  let acquisitionTax = 0;
  let educationTax = 0;
  let localTax = 0;
  let totalTax = 0;

  // 세율 정보 (2024년 기준)
  const taxRates = {
    apartment: {
      seoul: { rate: 0.004, education: 0.002, local: 0.001 },
      busan: { rate: 0.004, education: 0.002, local: 0.001 },
      daegu: { rate: 0.004, education: 0.002, local: 0.001 },
      incheon: { rate: 0.004, education: 0.002, local: 0.001 },
      gwangju: { rate: 0.004, education: 0.002, local: 0.001 },
      daejeon: { rate: 0.004, education: 0.002, local: 0.001 },
      ulsan: { rate: 0.004, education: 0.002, local: 0.001 },
      sejong: { rate: 0.004, education: 0.002, local: 0.001 },
      gyeonggi: { rate: 0.004, education: 0.002, local: 0.001 },
      other: { rate: 0.004, education: 0.002, local: 0.001 }
    },
    house: {
      seoul: { rate: 0.004, education: 0.002, local: 0.001 },
      busan: { rate: 0.004, education: 0.002, local: 0.001 },
      daegu: { rate: 0.004, education: 0.002, local: 0.001 },
      incheon: { rate: 0.004, education: 0.002, local: 0.001 },
      gwangju: { rate: 0.004, education: 0.002, local: 0.001 },
      daejeon: { rate: 0.004, education: 0.002, local: 0.001 },
      ulsan: { rate: 0.004, education: 0.002, local: 0.001 },
      sejong: { rate: 0.004, education: 0.002, local: 0.001 },
      gyeonggi: { rate: 0.004, education: 0.002, local: 0.001 },
      other: { rate: 0.004, education: 0.002, local: 0.001 }
    },
    land: {
      seoul: { rate: 0.004, education: 0.002, local: 0.001 },
      busan: { rate: 0.004, education: 0.002, local: 0.001 },
      daegu: { rate: 0.004, education: 0.002, local: 0.001 },
      incheon: { rate: 0.004, education: 0.002, local: 0.001 },
      gwangju: { rate: 0.004, education: 0.002, local: 0.001 },
      daejeon: { rate: 0.004, education: 0.002, local: 0.001 },
      ulsan: { rate: 0.004, education: 0.002, local: 0.001 },
      sejong: { rate: 0.004, education: 0.002, local: 0.001 },
      gyeonggi: { rate: 0.004, education: 0.002, local: 0.001 },
      other: { rate: 0.004, education: 0.002, local: 0.001 }
    }
  };

  // 지역 옵션
  const locations = [
    { value: 'seoul', label: '서울특별시' },
    { value: 'busan', label: '부산광역시' },
    { value: 'daegu', label: '대구광역시' },
    { value: 'incheon', label: '인천광역시' },
    { value: 'gwangju', label: '광주광역시' },
    { value: 'daejeon', label: '대전광역시' },
    { value: 'ulsan', label: '울산광역시' },
    { value: 'sejong', label: '세종특별자치시' },
    { value: 'gyeonggi', label: '경기도' },
    { value: 'other', label: '기타 지역' }
  ];

  // 부동산 유형 옵션
  const propertyTypes = [
    { value: 'apartment', label: '아파트' },
    { value: 'house', label: '단독주택' },
    { value: 'land', label: '토지' }
  ];

  // 세금 계산 함수
  function calculateTax() {
    if (propertyValue <= 0) {
      acquisitionTax = 0;
      educationTax = 0;
      localTax = 0;
      totalTax = 0;
      return;
    }

    const rates = taxRates[propertyType][location];
    let baseRate = rates.rate;

    // 첫 주택 감면 (50% 감면)
    if (isFirstHome && (propertyType === 'apartment' || propertyType === 'house')) {
      baseRate *= 0.5;
    }

    // 신축 감면 (50% 감면, 3년간)
    if (isNewConstruction && (propertyType === 'apartment' || propertyType === 'house')) {
      baseRate *= 0.5;
    }

    acquisitionTax = Math.round(propertyValue * baseRate);
    educationTax = Math.round(acquisitionTax * rates.education);
    localTax = Math.round(acquisitionTax * rates.local);
    totalTax = acquisitionTax + educationTax + localTax;
  }

  // 입력값 변경 시 자동 계산
  $: if (propertyValue > 0) {
    calculateTax();
  }
</script>

<div class="calculator">
  <h2>📥 취득세 계산기</h2>
  
  <div class="input-section">
    <div class="input-group">
      <label for="propertyValue">부동산 가액 (원)</label>
      <input 
        type="number" 
        id="propertyValue"
        bind:value={propertyValue}
        placeholder="예: 500000000"
        min="0"
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

    <div class="input-group">
      <label for="location">지역</label>
      <select id="location" bind:value={location}>
        {#each locations as loc}
          <option value={loc.value}>{loc.label}</option>
        {/each}
      </select>
    </div>

    <div class="input-group">
      <label for="acquisitionDate">취득일</label>
      <input 
        type="date" 
        id="acquisitionDate"
        bind:value={acquisitionDate}
      />
    </div>

    <div class="checkbox-group">
      <label class="checkbox-label">
        <input 
          type="checkbox" 
          bind:checked={isFirstHome}
        />
        첫 주택 (50% 감면)
      </label>
    </div>

    <div class="checkbox-group">
      <label class="checkbox-label">
        <input 
          type="checkbox" 
          bind:checked={isNewConstruction}
        />
        신축 (50% 감면, 3년간)
      </label>
    </div>
  </div>

  {#if propertyValue > 0}
    <div class="result-section">
      <h3>📊 계산 결과</h3>
      
      <div class="result-grid">
        <div class="result-item">
          <span class="result-label">취득세</span>
          <span class="result-value">{acquisitionTax.toLocaleString()}원</span>
        </div>
        
        <div class="result-item">
          <span class="result-label">교육세</span>
          <span class="result-value">{educationTax.toLocaleString()}원</span>
        </div>
        
        <div class="result-item">
          <span class="result-label">지방교육세</span>
          <span class="result-value">{localTax.toLocaleString()}원</span>
        </div>
        
        <div class="result-item total">
          <span class="result-label">총 세금</span>
          <span class="result-value">{totalTax.toLocaleString()}원</span>
        </div>
      </div>

      <div class="tax-rate-info">
        <h4>💡 세율 정보</h4>
        <p>• 취득세: {taxRates[propertyType][location].rate * 100}%</p>
        <p>• 교육세: 취득세의 {taxRates[propertyType][location].education * 100}%</p>
        <p>• 지방교육세: 취득세의 {taxRates[propertyType][location].local * 100}%</p>
        {#if isFirstHome}
          <p class="discount">• 첫 주택 감면 적용 (50% 감면)</p>
        {/if}
        {#if isNewConstruction}
          <p class="discount">• 신축 감면 적용 (50% 감면)</p>
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
    border-color: #3498db;
    box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.1);
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
    background: #3498db;
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

  .tax-rate-info {
    background: white;
    padding: 20px;
    border-radius: 8px;
    border-left: 4px solid #3498db;
    border: 1px solid #e0e0e0;
  }

  .tax-rate-info h4 {
    color: #1a252f;
    margin-bottom: 15px;
    font-size: 1.1rem;
    font-weight: 700;
  }

  .tax-rate-info p {
    margin: 8px 0;
    color: #2c3e50;
    font-size: 1rem;
    font-weight: 500;
  }

  .discount {
    color: #27ae60 !important;
    font-weight: 700;
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
