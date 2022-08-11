<template>
  <main>
    <div class="left">
      <section class="inputs">
        <form>
          <h2>Basisdaten</h2>
          <div>
            <label for="price">Preis (€)</label>
            <input
              type="number"
              name="price"
              v-model.number="basicData.price"
            />
          </div>
          <div>
            <label for="qm">Vermietbare Fläche (qm)</label>
            <input
              type="number"
              name="qm"
              v-model.number="basicData.rentableSpace"
            />
          </div>
          <div>
            <label for="rent-month">Kaltmiete/ Monat (€)</label>
            <input
              type="number"
              name="rent-month"
              autocomplete="off"
              v-model.number="basicData.rentPerMonth"
            />
          </div>
          <div>
            <label for="rent-year">Kaltmiete/ Jahr (€)</label>
            <input
              type="number"
              name="rent-year"
              autocomplete="off"
              v-model="calculateRent"
            />
          </div>
        </form>
      </section>

      <section class="res">
        <h2>Rentabilität</h2>
        <table>
          <tbody>
            <tr>
              <th>Nettorendite</th>
              <td>{{ renderNetRateOfReturn }}</td>
            </tr>
            <tr>
              <th>Faktor</th>
              <td>{{ rentabilityFactor }}</td>
            </tr>
            <tr>
              <th>Cashflow</th>
              <td :class="[isCashflowPositiv ? 'profit' : 'no-profit']">
                {{ renderCashflow }}
              </td>
            </tr>
          </tbody>
        </table>
      </section>

      <section class="inputs">
        <form>
          <h2>Ergänzende Daten</h2>
          <div>
            <label for="allowance"> Hausgeld/ Monat (€) </label>
            <input
              type="number"
              name="allowance"
              v-model.number="costParameters.allowance"
            />
          </div>
          <div>
            <label for="repair-savings">Instandhaltungsrücklage (€ /qm)</label>
            <input
              type="number"
              name="repair-savings"
              v-model.number="costParameters.rateForRepairSavings"
            />
          </div>
          <div>
            <label for="rent-loss">kalk. Mietausfall/ Monat (%)</label>
            <input
              type="number"
              name="rent-loss"
              v-model.number="costParameters.rentLossRatio"
            />
          </div>
        </form>
      </section>

      <section class="inputs">
        <form>
          <h2>Finanzierung</h2>
          <div>
            <label for="interest-rate">Zins (%)</label>
            <input
              type="number"
              name="interest-rate"
              v-model.number="financing.interestRate"
            />
          </div>
          <div>
            <label for="amortization">Tilgung (%)</label>
            <input
              type="number"
              name="amortization"
              v-model.number="financing.amortization"
            />
          </div>
        </form>
      </section>

      <section class="inputs">
        <form>
          <h2>Einmalige Kosten</h2>
          <div>
            <label for="realtor">Maklerprovision (%)</label>
            <input
              type="number"
              name="realtor"
              v-model.number="oneTimeCosts.realtorCommission"
            />
          </div>
        </form>
      </section>
    </div>
    <div class="right">
      <section class="res">
        <h2>Pro qm</h2>
        <table>
          <tbody>
            <tr>
              <th>Miete pro qm</th>
              <td>{{ renderRentPerSpace }}</td>
            </tr>
            <tr>
              <th>Preis pro qm</th>
              <td>{{ renderPricePerSpace }}</td>
            </tr>
          </tbody>
        </table>
        <h2>Mieten</h2>
        <table>
          <thead>
            <tr>
              <th></th>
              <th>Monat</th>
              <th>Jahr</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <th>Kaltmiete pro Monat</th>
              <td>{{ renderRentPerMonth }}</td>
              <td>{{ renderRentPerYear }}</td>
            </tr>
            <tr>
              <th>Warmmiete pro Monat</th>
              <td>{{ renderTotalRentPerMonth }}</td>
              <td>{{ renderTotalRentPerYear }}</td>
            </tr>
          </tbody>
        </table>
        <h2>Kosten</h2>
        <table>
          <thead>
            <tr>
              <th></th>
              <th>Monat</th>
              <th>Jahr</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <th>Zinsen</th>
              <td>{{ renderMonthlyInterests }}</td>
              <td>{{ renderAnnualInterests }}</td>
            </tr>
            <tr>
              <th>Tilgung</th>
              <td>{{ renderMonthlyAmortization }}</td>
              <td>{{ renderAnnualAmortization }}</td>
            </tr>
            <tr class="subtotal">
              <th>Kapitaldienst</th>
              <td>{{ renderMonthlyDebtService }}</td>
              <td>{{ renderAnnuity }}</td>
            </tr>
            <tr>
              <th>Instandhaltungskosten</th>
              <td>{{ renderRepairSavings }}</td>
              <td>{{ formatCurrency(repairSavings * 12) }}</td>
            </tr>
            <tr>
              <th>Kalkulatorischer Mietausfall</th>
              <td>{{ formatCurrency(rentLossSavings) }}</td>
              <td>{{ formatCurrency(rentLossSavings * 12) }}</td>
            </tr>
            <tr class="subtotal">
              <th>Bewirtschaftungskosten</th>
              <td>{{ formatCurrency(monthlyOperatingCosts) }}</td>
              <td>{{ formatCurrency(monthlyOperatingCosts * 12) }}</td>
            </tr>
            <tr class="total">
              <th>Summe</th>
              <td>{{ formatCurrency(monthlyCosts) }}</td>
              <td>{{ formatCurrency(monthlyCosts * 12) }}</td>
            </tr>
          </tbody>
        </table>
      </section>
    </div>
  </main>
</template>

<script>
export default {
  name: "Basic-Calculations",
  data() {
    return {
      basicData: {
        price: 1090000,
        rentPerMonth: 1000,
        rentPerYear: 12000,
        rentableSpace: 80,
      },
      costParameters: {
        allowance: 0,
        rentLossRatio: 0,
        rateForRepairSavings: 0,
      },
      oneTimeCosts: {
        realtorCommission: 3.57,
      },
      financing: {
        interestRate: 3,
        amortization: 2,
      },
    };
  },
  computed: {
    calculateRent: {
      get() {
        return this.setRent();
      },
      set(newValue) {
        this.basicData.rentPerMonth = newValue / 12;
      },
    },
    renderRentPerMonth() {
      return this.formatCurrency(this.basicData.rentPerMonth);
    },
    renderRentPerYear() {
      return this.formatCurrency(this.basicData.rentPerYear);
    },
    netRateOfReturn() {
      return Math.round(
        (this.basicData.rentPerYear / this.basicData.price) * 100
      );
    },
    renderNetRateOfReturn() {
      return `${this.netRateOfReturn} %`;
    },
    rentabilityFactor() {
      return this.basicData.price > 0 && this.basicData.rentPerYear > 0
        ? Math.round(this.basicData.price / this.basicData.rentPerYear)
        : 0;
    },
    cashflow() {
      return this.basicData.rentPerMonth > 0
        ? this.basicData.rentPerMonth - this.monthlyCosts
        : 0;
    },
    isCashflowPositiv() {
      return this.cashflow > 0 ? true : false;
    },
    renderCashflow() {
      return this.formatCurrency(this.cashflow);
    },
    monthlyInterests() {
      return (this.basicData.price * (this.financing.interestRate / 100)) / 12;
    },
    renderMonthlyInterests() {
      return this.formatCurrency(this.monthlyInterests);
    },
    annualInterests() {
      return this.basicData.price * (this.financing.interestRate / 100);
    },
    renderAnnualInterests() {
      return this.formatCurrency(this.annualInterests);
    },
    monthlyAmortization() {
      return (this.basicData.price * (this.financing.amortization / 100)) / 12;
    },
    renderMonthlyAmortization() {
      return this.formatCurrency(this.monthlyAmortization);
    },
    annualAmortization() {
      return this.basicData.price * (this.financing.amortization / 100);
    },
    renderAnnualAmortization() {
      return this.formatCurrency(this.annualAmortization);
    },
    annuity() {
      return this.annualInterests + this.annualAmortization;
    },
    renderAnnuity() {
      return this.formatCurrency(this.annuity);
    },
    monthlyDebtService() {
      return this.annuity / 12;
    },
    renderMonthlyDebtService() {
      return this.formatCurrency(this.monthlyDebtService);
    },
    repairSavings() {
      return (
        (this.basicData.rentableSpace *
          this.costParameters.rateForRepairSavings) /
        12
      );
    },
    renderRepairSavings() {
      return this.formatCurrency(this.repairSavings);
    },
    rentLossSavings() {
      return (
        (this.costParameters.rentLossRatio / 100) * this.basicData.rentPerMonth
      );
    },
    monthlyOperatingCosts() {
      return (
        this.repairSavings +
        this.rentLossSavings +
        this.costParameters.allowance
      );
    },
    monthlyCosts() {
      return this.monthlyOperatingCosts + this.monthlyDebtService;
    },
    rentPerSpace() {
      return this.basicData.rentPerMonth / this.basicData.rentableSpace;
    },
    renderRentPerSpace() {
      return this.formatCurrency(this.rentPerSpace);
    },
    pricePerSpace() {
      return this.basicData.rentableSpace === 0 ||
        this.basicData.rentableSpace === undefined
        ? 0
        : this.basicData.price / this.basicData.rentableSpace;
    },
    renderPricePerSpace() {
      return this.formatCurrency(this.pricePerSpace);
    },
    totalRentPerMonth() {
      return this.basicData.rentPerMonth + this.costParameters.allowance;
    },
    totalRentPerYear() {
      return this.totalRentPerMonth * 12;
    },
    renderTotalRentPerMonth() {
      return this.formatCurrency(this.totalRentPerMonth);
    },
    renderTotalRentPerYear() {
      return this.formatCurrency(this.totalRentPerYear);
    },
  },

  methods: {
    formatCurrency(amount) {
      return Intl.NumberFormat("de-DE", {
        style: "currency",
        currency: "EUR",
      }).format(amount);
    },
    setRent() {
      let yearlyRent = this.basicData.rentPerMonth * 12;
      this.basicData.rentPerYear = yearlyRent;
      return yearlyRent;
    },
  },

  /* Rentabilität -----------------
    Mietrendite = Miete pro Jahr / Kaufpreis  (umgekehrt zum Faktor)
    6 % = Faktor 17
  */
};
</script>

<style lang="scss" scoped>
main {
  width: 80vw;
  margin: 50px auto;
  display: flex;
  gap: 1rem;
}
.left,
.right {
  width: 50%;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
section {
  border-radius: 5px;
  padding: 15px;
}
.inputs {
  background: #f4f4f4;
  color: black;
}
.res {
  background: darkslategray;
  color: white;
}
h2 {
  text-align: left;
  padding-left: 10px;
}
h2,
h3 {
  color: #2f8f9d;
}
.res h2,
.res h3 {
  color: orangered;
}
form {
  display: flex;
  gap: 0.6rem;
  flex-direction: column;
}
form div {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}
label {
  margin-right: 20px;
  font-size: 14pt;
  padding: 10px;
  font-weight: bold;
}
input {
  height: 25px;
  width: 35%;
  padding: 10px;
  font-size: 14pt;
  text-align: center;
  border: 1px solid #ccc;
  border-radius: 5px;
}
.calculated {
  background: red;
}
hr {
  margin: 40px 0 20px 0;
}
table {
  width: 96%;
  margin: 0 auto;
}
table,
th,
td {
  border: 1px solid lightgray;
  border-collapse: collapse;
}
.subtotal,
.subtotal td {
  color: orangered;
}
.total,
.total td {
  background-color: orangered;
}
tbody th {
  width: 50%;
  text-align: left;
}
thead th {
  background: #234343;
}
th {
  padding: 15px;
}
td {
  padding: 15px;
  /* background: #234343; */
  color: #fff;
  font-weight: bold;
  text-align: right;
}
.profit {
  background: #5fd068;
  color: black;
}
.no-profit {
  background: #eb1d36;
  color: white;
}
</style>
