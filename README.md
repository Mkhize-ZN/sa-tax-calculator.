'use client';

import React, { useState } from 'react';

export default function ComprehensiveTaxCalculator() {
  const [grossIncome, setGrossIncome] = useState<number>(450000);
  const [travelAllowance, setTravelAllowance] = useState<number>(0);
  const [ageGroup, setAgeGroup] = useState<number>(1); // 1 = under 65, 2 = 65-74, 3 = 75+
  const [raContribution, setRaContribution] = useState<number>(0);
  const [dependents, setDependents] = useState<number>(0);
  const [outOfPocketMedical, setOutOfPocketMedical] = useState<number>(0);

  // --- SARS CALCULATIONS (2026/2027 TAX YEAR) ---

  // 1. Travel Allowance: 80% is subject to PAYE standard inclusion
  const taxableTravel = travelAllowance * 0.80;
  const totalRemuneration = grossIncome + taxableTravel;

  // 2. Retirement Annuity Deduction Cap (27.5% of remuneration or R430,000 max)
  const maxRaAllowed = Math.min(totalRemuneration * 0.275, 430000);
  const actualRa = Math.min(raContribution, maxRaAllowed);
  
  const taxableIncome = Math.max(0, totalRemuneration - actualRa);

  // 3. Tax Brackets
  let annualTaxBeforeRebate = 0;
  if (taxableIncome <= 245100) {
    annualTaxBeforeRebate = taxableIncome * 0.18;
  } else if (taxableIncome <= 383100) {
    annualTaxBeforeRebate = 44118 + (taxableIncome - 245100) * 0.26;
  } else if (taxableIncome <= 530200) {
    annualTaxBeforeRebate = 79998 + (taxableIncome - 383100) * 0.31;
  } else if (taxableIncome <= 695800) {
    annualTaxBeforeRebate = 125599 + (taxableIncome - 530200) * 0.36;
  } else if (taxableIncome <= 887000) {
    annualTaxBeforeRebate = 185215 + (taxableIncome - 695800) * 0.39;
  } else if (taxableIncome <= 1878600) {
    annualTaxBeforeRebate = 259783 + (taxableIncome - 887000) * 0.41;
  } else {
    annualTaxBeforeRebate = 666339 + (taxableIncome - 1878600) * 0.45;
  }

  // 4. Age Rebates
  let rebate = 17820; // Primary
  if (ageGroup === 2) rebate += 9765; // Secondary (65+)
  if (ageGroup === 3) rebate += 9765 + 3249; // Tertiary (75+)

  let taxAfterRebate = Math.max(0, annualTaxBeforeRebate - rebate);

  // 5. Medical Scheme Fees Tax Credits (Section 6A)
  // R376/mo for main, R376/mo for 1st dep, R254/mo for each extra
  let annualMedicalCredit = 0;
  if (dependents > 0) {
    const tier1Count = Math.min(dependents, 2);
    const extraCount = Math.max(0, dependents - 2);
    annualMedicalCredit = (tier1Count * 376 * 12) + (extraCount * 254 * 12);
  }

  // 6. Additional Medical Expenses Credit (Section 6B)
  let additionalMedicalCredit = 0;
  const annualSchemeCreditForThreshold = annualMedicalCredit;
  
  if (ageGroup >= 2) {
    // 65 and older: 33.3% of [out-of-pocket + medical scheme fees exceeding 3x annual credits]
    // Simplified calculation wrapper for extra medical relief
    additionalMedicalCredit = outOfPocketMedical * 0.333;
  } else {
    // Under 65: 25% of [out-of-pocket exceeding (4x annual scheme credits + 7.5% of taxable income)]
    const threshold = (4 * annualSchemeCreditForThreshold) + (0.075 * taxableIncome);
    const excessMedical = Math.max(0, outOfPocketMedical - threshold);
    additionalMedicalCredit = excessMedical * 0.25;
  }

  const totalMedicalCredits = annualMedicalCredit + additionalMedicalCredit;
  const finalAnnualTax = Math.max(0, taxAfterRebate - totalMedicalCredits);
  
  const finalMonthlyTax = finalAnnualTax / 12;
  const totalMonthlyGross = (grossIncome + travelAllowance) / 12;
  const monthlyNet = totalMonthlyGross - finalMonthlyTax - (actualRa / 12);
  const effectiveRate = totalRemuneration > 0 ? (finalAnnualTax / totalRemuneration) * 100 : 0;

  return (
    <main className="min-h-screen bg-slate-50 py-10 px-4 text-slate-800">
      <div className="max-w-2xl mx-auto bg-white rounded-2xl shadow-xl p-6 md:p-8 border border-slate-100">
        <h1 className="text-2xl font-bold text-slate-900 mb-2">Comprehensive SA Tax & Take-Home Calculator</h1>
        <p className="text-sm text-slate-500 mb-6">Includes travel allowances, retirement annuities, and medical credits.</p>

        <div className="space-y-4">
          {/* Gross Income Input */}
          <div>
            <label className="block text-xs font-semibold uppercase tracking-wider text-slate-600 mb-1">
              Annual Gross Salary / Earnings (Rands)
            </label>
            <input 
              type="number" 
              value={grossIncome} 
              onChange={(e) => setGrossIncome(Number(e.target.value))}
              className="w-full px-4 py-3 rounded-lg border border-slate-200 focus:ring-2 focus:ring-blue-500 focus:outline-none font-medium"
            />
          </div>

          {/* Travel Allowance */}
          <div>
            <label className="block text-xs font-semibold uppercase tracking-wider text-slate-600 mb-1">
              Annual Travel Allowance (Rands) <span className="text-slate-400 font-normal">({'(80% taxable)'})</span>
            </label>
            <input 
              type="number" 
              value={travelAllowance} 
              onChange={(e) => setTravelAllowance(Number(e.target.value))}
              className="w-full px-4 py-3 rounded-lg border border-slate-200 focus:ring-2 focus:ring-blue-500 focus:outline-none"
            />
          </div>

          {/* Age Group Selector */}
          <div>
            <label className="block text-xs font-semibold uppercase tracking-wider text-slate-600 mb-1">
              Age Group (Rebates & Medical Thresholds)
            </label>
            <select 
              value={ageGroup} 
              onChange={(e) => setAgeGroup(Number(e.target.value))}
              className="w-full px-4 py-3 rounded-lg border border-slate-200 focus:ring-2 focus:ring-blue-500 focus:outline-none bg-white"
            >
              <option value={1}>Under 65 years</option>
              <option value={2}>65 to 74 years</option>
              <option value={3}>75 years and older</option>
            </select>
          </div>

          {/* Retirement Annuity */}
          <div>
            <label className="block text-xs font-semibold uppercase tracking-wider text-slate-600 mb-1">
              Annual Retirement Annuity (RA) Contribution
            </label>
            <input 
              type="number" 
              value={raContribution} 
              onChange={(e) => setRaContribution(Number(e.target.value))}
              className="w-full px-4 py-3 rounded-lg border border-slate-200 focus:ring-2 focus:ring-blue-500 focus:outline-none"
            />
          </div>

          {/* Medical Aid Members */}
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div>
              <label className="block text-xs font-semibold uppercase tracking-wider text-slate-600 mb-1">
                Total Medical Aid Members
              </label>
              <input 
                type="number" 
                value={dependents} 
                onChange={(e) => setDependents(Number(e.target.value))}
                className="w-full px-4 py-3 rounded-lg border border-slate-200 focus:ring-2 focus:ring-blue-500 focus:outline-none"
              />
            </div>
            <div>
              <label className="block text-xs font-semibold uppercase tracking-wider text-slate-600 mb-1">
                Annual Out-of-Pocket Medical Costs
              </label>
              <input 
                type="number" 
                value={outOfPocketMedical} 
                onChange={(e) => setOutOfPocketMedical(Number(e.target.value))}
                className="w-full px-4 py-3 rounded-lg border border-slate-200 focus:ring-2 focus:ring-blue-500 focus:outline-none"
              />
            </div>
          </div>
        </div>

        {/* Results Card */}
        <div className="mt-8 bg-slate-900 text-white rounded-xl p-6 shadow-inner">
          <h2 className="text-xs font-semibold uppercase tracking-widest text-blue-400 mb-4">Tax & Cash Flow Summary</h2>
          
          <div className="grid grid-cols-2 gap-4 border-b border-slate-800 pb-4 mb-4">
            <div>
              <p className="text-xs text-slate-400">Total Monthly Gross</p>
              <p className="text-xl font-bold">R {totalMonthlyGross.toLocaleString('en-ZA', { maximumFractionDigits: 0 })}</p>
            </div>
            <div>
              <p className="text-xs text-slate-400">Monthly Net Take-Home</p>
              <p className="text-2xl font-extrabold text-emerald-400">R {monthlyNet.toLocaleString('en-ZA', { maximumFractionDigits: 0 })}</p>
            </div>
          </div>

          <div className="space-y-2 text-sm text-slate-300">
            <div className="flex justify-between">
              <span>Taxable Income (After RA):</span>
              <span className="font-semibold text-white">R {taxableIncome.toLocaleString('en-ZA', { maximumFractionDigits: 0 })}</span>
            </div>
            <div className="flex justify-between">
              <span>Final Annual Tax (PAYE):</span>
              <span className="font-semibold text-white">R {finalAnnualTax.toLocaleString('en-ZA', { maximumFractionDigits: 0 })}</span>
            </div>
            <div className="flex justify-between">
              <span>Total Medical Tax Credits Applied:</span>
              <span className="font-semibold text-emerald-400">- R {totalMedicalCredits.toLocaleString('en-ZA', { maximumFractionDigits: 0 })}</span>
            </div>
            <div className="flex justify-between border-t border-slate-800 pt-2 mt-2">
              <span>Effective Tax Rate:</span>
              <span className="font-semibold text-white">{effectiveRate.toFixed(1)}%</span>
            </div>
          </div>
        </div>
      </div>
    </main>
  );
}
