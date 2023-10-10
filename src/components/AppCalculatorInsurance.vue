<template>
  <main class="calculator">
    <h1>Kalkulačka cestovného poistenia</h1>
    <form @submit.prevent="calculateInsurance" class="form">
      <!-- Variant poistenia -->
      <div class="form-group">
        <label for="insuranceType">Variant poistenia:</label>
        <select
          v-model="selectedInsuranceType"
          id="insuranceType"
          class="select"
          required
        >
          <option value="kratkodobe">Krátkodobé poistenie</option>
          <option value="celorocne">Celorocné poistenie</option>
        </select>
      </div>

      <!-- Začiatok poistenia -->
      <div class="form-group">
        <label for="startDate">Začiatok poistenia:</label>
        <input
          type="date"
          v-model="startDate"
          id="startDate"
          required
          class="input"
        />
      </div>

      <!-- Koniec poistenia -->
      <div class="form-group">
        <label for="endDate">Koniec poistenia:</label>
        <input
          type="date"
          v-model="endDate"
          id="endDate"
          :required="selectedInsuranceType === 'kratkodobe'"
          class="input"
        />
      </div>

      <!-- Balík -->
      <div class="form-group">
        <label for="package">Balík:</label>
        <select v-model="selectedPackage" id="package" class="select" required>
          <option value="zakladny">Základný</option>
          <option value="rozsireny">Rozšírený</option>
          <option value="extra">Extra</option>
        </select>
      </div>

      <!-- Pripoistenia -->
      <div class="form-group">
        <label for="additionalInsurances">Pripoistenia:</label>
        <div class="checkbox-label">
          <input
            type="checkbox"
            v-model="stornoCesty"
            id="stornoCesty"
            class="checkbox"
          />
          <label class="pripoistenie" for="stornoCesty">Storno cesty</label>
        </div>
        <div class="checkbox-label">
          <input
            type="checkbox"
            v-model="sportoveAktivity"
            id="sportoveAktivity"
            class="checkbox"
          />
          <label class="pripoistenie" for="sportoveAktivity"
            >Športové aktivity</label
          >
        </div>
      </div>

      <!-- Počet osôb -->
      <div class="form-group">
        <label for="numberOfPersons">Počet osôb:</label>
        <select
          v-model="numberOfPersons"
          id="numberOfPersons"
          required
          class="select"
        >
          <option value="1">1 osoba</option>
          <option value="2">2 osoby</option>
          <option value="3">3 osoby</option>
        </select>
      </div>

      <!-- Tlačítko na výpočet ceny poistenia -->
      <div class="form-group">
        <button class="button" type="submit">Vypočítať cenu poistenia</button>
      </div>
    </form>
    <!-- Vysledok -->
    <transition name="fade" mode="out-in">
      <calculator-result :insurancePrice="insurancePrice" class="result" />
    </transition>
  </main>
</template>

<script>
import CalculatorResult from "./CalculatorResult.vue";
export default {
  name: "AppCalculatorInsurance",
  components: { CalculatorResult },
  data() {
    return {
      selectedInsuranceType: null, // Typ poistenia (krátkodobé alebo celoročné)
      startDate: new Date().toISOString().split("T")[0], // Dátum začiatku poistenia je vzdy aktualny
      endDate: null, // Dátum konca poistenia
      selectedPackage: null, // Vybraný balík poistenia
      stornoCesty: false, // Pripoistenie - Storno cesty (true ak je začiarknuté, inak false)
      sportoveAktivity: false, // Pripoistenie - Športové aktivity (true ak je začiarknuté, inak false)
      numberOfPersons: 0, // Počet osôb poistených
      insurancePrice: null, // Cena poistenia
      minAnnualEndDate: null, // Inicializujeme minAnnualEndDate na null
    };
  },
  methods: {
    calculateInsurance() {
      // Sadzby pre krátkodobé a celoročné poistenie a balíky
      const shortTermRates = {
        zakladny: 1.2,
        rozsireny: 1.8,
        extra: 2.4,
      };

      const annualRates = {
        zakladny: 39,
        rozsireny: 49,
        extra: 59,
      };

      // Prirážky pre pripoistenia
      const stornoPrirazka = this.stornoCesty ? 0.2 : 0; // Prirážka pre storno cesty (20% začiarknuté, inak 0%)
      const sportoveAktivityPrirazka = this.sportoveAktivity ? 0.1 : 0; // Prirážka pre športové aktivity (10% začiarknuté, inak 0%)

      // Prirážky pre krátkodobé poistenie
      const shortTermStornoPrirazka = this.stornoCesty ? 0.5 : 0; // Prirážka pre storno cesty (50%)
      const shortTermSportoveAktivityPrirazka = this.sportoveAktivity ? 0.3 : 0; // Prirážka pre športové aktivity (30%)

      // Počet milisekund v jednom dni
      const oneDay = 24 * 60 * 60 * 1000;

      if (this.selectedInsuranceType === "celorocne") {
        // Výpočet ceny pre celoročne poistenia
        this.insurancePrice =
          annualRates[this.selectedPackage] + // Cena za vybraný balík poistenia na den
          this.numberOfPersons * // Počet poistenych osob
            (stornoPrirazka * annualRates[this.selectedPackage]) + // Prirážka pre storno cesty
          sportoveAktivityPrirazka * annualRates[this.selectedPackage]; // Prirážka pre športové aktivity
      } else {
        // Výpočet ceny pre krátkodobé poistenia
        let selectedRates = null;
        let selectedStornoPrirazka = shortTermStornoPrirazka;
        let selectedSportoveAktivityPrirazka =
          shortTermSportoveAktivityPrirazka;

        if (this.selectedInsuranceType === "kratkodobe") {
          selectedRates = shortTermRates;
        }

        const baseRate = selectedRates[this.selectedPackage]; // Sadzba za vybraný balík poistenia
        const numberOfDays =
          (new Date(this.endDate) - new Date(this.startDate)) / oneDay; // Počet dní poistenia
        const totalPrice =
          baseRate * // Cena za vybraný balík poistenia na den
          numberOfDays * // Celkový počet dní poistenia
          this.numberOfPersons * // Počet poistenych osob
          (1 + selectedStornoPrirazka + selectedSportoveAktivityPrirazka); // Prirážky pre storno a športové aktivity

        this.insurancePrice = totalPrice; // Nastavení vypočítané ceny poistenia
      }
    },
    handleInsuranceTypeChange() {
      // Ak je vybraný "Celorocné poistenie," nastavíme koncový dátum na rok od aktuálneho dátumu
      if (this.selectedInsuranceType === "celorocne") {
        this.endDate = new Date(new Date().getFullYear() + 1, 0, 1)
          .toISOString()
          .split("T")[0];
      }
    },
  },
  watch: {
    selectedInsuranceType(newVal) {
      if (newVal === "celorocne") {
        // Ak je vybrané "Celorocné poistenie," nastavíme koncový dátum na 365 dní od aktuálneho dátumu
        const currentDate = new Date();
        const endDate = new Date(currentDate);
        endDate.setDate(currentDate.getDate() + 365);
        this.endDate = endDate.toISOString().split("T")[0];
      }
    },
  },
};
</script>


<style>
.calculator {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: #f0f0f0;
  padding: 20px;
}

h1 {
  font-size: 24px;
  margin-bottom: 20px;
}

.form {
  display: grid;
  justify-items: center;
  align-items: center;
  width: 100%;
  max-width: 400px;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #fff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin: 10px 0;
  display: grid;
  justify-items: flex-start;
  align-content: center;
  width: 100%;
}

.label {
  font-weight: bold;
  margin-bottom: 5px;
}

.input,
.select {
  width: 100%;
  padding: 10px 0;
  border: 1px solid #ccc;
  border-radius: 3px;
  background-color: #f8f8f8;
  font-size: 20px;
  text-align: center;
  cursor: pointer;
}

.checkbox-label {
  display: flex;
  align-items: center;
  font-size: 16px;
  margin-top: 5px;
  font-weight: normal;
}

.checkbox {
  margin-right: 5px;
  width: 20px;
  height: 20px;
  cursor: pointer;
}
.pripoistenie {
  font-weight: normal;
  font-size: 18px;
}
.button {
  width: 100%;
  padding: 20px;
  background-color: #f99a34;
  color: #fff;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  transition: background-color 0.3s ease;
  margin-top: 40px;
  font-size: 24px;
  text-transform: uppercase;
  font-weight: bold;
}

.button:hover {
  background-color: #0056b3;
}

.button:active {
  transform: scale(0.95);
}

#startDate {
  color: green;
}

#endDate {
  color: red;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}

label {
  font-size: 22px;
  font-weight: bold;
}
</style>

