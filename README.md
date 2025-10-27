import React, { useState } from "react";

// KDA Problem Solver - Single-file React component
// Tailwind CSS utility classes are used for styling (no import required when used in environments with Tailwind configured).
// Default export so this component can be previewed directly.

export default function KdaProblemSolver() {
  const [kills, setKills] = useState(10);
  const [deaths, setDeaths] = useState(2);
  const [assists, setAssists] = useState(5);
  const [desiredKda, setDesiredKda] = useState(5);
  const [simMatches, setSimMatches] = useState(3);
  const [history, setHistory] = useState([]);

  // compute KDA: many communities use (kills + assists) / max(1, deaths) to avoid division by zero
  function computeKda(k, d, a) {
    const denom = Math.max(1, Number(d));
    return (Number(k) + Number(a)) / denom;
  }

  function formatNum(n) {
    return Number(n).toFixed(2);
  }

  function calculate() {
    const kda = computeKda(kills, deaths, assists);
    return formatNum(kda);
  }

  // Suggest actions to reach a desired KDA in one match (approximate)
  function suggestionForTarget(target) {
    const d = Math.max(1, Number(deaths));
    const currentContrib = Number(assists) + Number(kills);
    // We want (newKills + newAssists) / d >= target -> newKills + newAssists >= target * d
    const requiredContrib = target * d;
    const missing = requiredContrib - currentContrib;
    if (missing <= 0) return { message: "You're already at or above the desired KDA!", needKills: 0, needAssists: 0 };
    // sensible split: prioritize kills (rough guidance)
    const needKills = Math.ceil(Math.max(0, missing - 0.6 * Number(assists)));
    const needAssists = Math.ceil(Math.max(0, missing - needKills));
    return { message: `You need roughly +${needKills} kills and +${needAssists} assists (approx).`, needKills, needAssists };
  }

  function runSimulation() {
    const sims = [];
    for (let i = 0; i < simMatches; i++) {
      // simple simulation: small random variance around current values
      const k = Math.max(0, Math.round(Number(kills) + (Math.random() - 0.3) * 3));
      const d = Math.max(0, Math.round(Number(deaths) + (Math.random() - 0.5) * 2));
      const a = Math.max(0, Math.round(Number(assists) + (Math.random() - 0.2) * 4));
      sims.push({ k, d, a, kda: computeKda(k, d, a) });
    }
    setHistory(sims);
  }

  function resetHistory() {
    setHistory([]);
  }

  // Helpful derived values
  const currentKda = computeKda(kills, deaths, assists);
  const targetHint = suggestionForTarget(Number(desiredKda));

  return (
    <div className="p-6 max-w-4xl mx-auto">
      <header className="mb-6">
        <h1 className="text-3xl font-extrabold">KDA Problem Solver</h1>
        <p className="text-sm text-gray-600 mt-1">Calculate, simulate, and get simple suggestions to reach your desired KDA.</p>
      </header>

      <main className="grid grid-cols-1 md:grid-cols-2 gap-6">
        <section className="bg-white rounded-2xl p-6 shadow">
          <h2 className="text-xl font-semibold mb-4">Inputs</h2>

          <label className="block mb-3">
            <div className="text-sm font-medium">Kills</div>
            <input type="number" value={kills} onChange={(e) => setKills(Math.max(0, Number(e.target.value)))} className="mt-1 block w-full rounded-md border p-2" />
          </label>

          <label className="block mb-3">
            <div className="text-sm font-medium">Deaths</div>
            <input type="number" value={deaths} onChange={(e) => setDeaths(Math.max(0, Number(e.target.value)))} className="mt-1 block w-full rounded-md border p-2" />
          </label>

          <label className="block mb-3">
            <div className="text-sm font-medium">Assists</div>
            <input type="number" value={assists} onChange={(e) => setAssists(Math.max(0, Number(e.target.value)))} className="mt-1 block w-full rounded-md border p-2" />
          </label>

          <div className="mt-4">
            <div className="text-sm text-gray-700">Current KDA</div>
            <div className="text-2xl font-bold">{formatNum(currentKda)} : 1</div>
          </div>

          <div className="mt-6 flex gap-3">
            <button onClick={() => { navigator.clipboard?.writeText(formatNum(currentKda)); }} className="px-4 py-2 rounded-lg bg-indigo-600 text-white">Copy KDA</button>
            <button onClick={() => { setKills(0); setDeaths(0); setAssists(0); }} className="px-4 py-2 rounded-lg border">Reset</button>
          </div>
        </section>

        <section className="bg-white rounded-2xl p-6 shadow">
          <h2 className="text-xl font-semibold mb-4">Goal Planner</h2>

          <label className="block mb-3">
            <div className="text-sm font-medium">Desired KDA</div>
            <input type="number" step="0.1" value={desiredKda} onChange={(e) => setDesiredKda(Math.max(0, Number(e.target.value)))} className="mt-1 block w-full rounded-md border p-2" />
          </label>

          <div className="mt-3">
            <div className="text-sm text-gray-700">Suggestion</div>
            <div className="mt-2 p-3 rounded-lg bg-gray-50">
              <div className="font-medium">{targetHint.message}</div>
              {targetHint.needKills > 0 && (
                <div className="text-sm text-gray-600 mt-1">Estimated extra kills: {targetHint.needKills}</div>
              )}
              {targetHint.needAssists > 0 && (
                <div className="text-sm text-gray-600">Estimated extra assists: {targetHint.needAssists}</div>
              )}
            </div>
          </div>

          <div className="mt-6">
            <h3 className="text-sm font-medium">Quick what-if</h3>
            <div className="mt-2 text-sm text-gray-700">If you change your match to the following numbers, it'll show the resulting KDA.</div>
            <div className="mt-3 grid grid-cols-2 gap-2">
              <button onClick={() => { setKills(kills + 3); }} className="px-3 py-2 rounded-lg border">+3 kills</button>
              <button onClick={() => { setAssists(assists + 5); }} className="px-3 py-2 rounded-lg border">+5 assists</button>
              <button onClick={() => { setDeaths(Math.max(0, deaths - 1)); }} className="px-3 py-2 rounded-lg border">-1 death</button>
              <button onClick={() => { setKills(0); setDeaths(0); setAssists(0); }} className="px-3 py-2 rounded-lg border">Clear</button>
            </div>
          </div>
        </section>

        <section className="md:col-span-2 bg-white rounded-2xl p-6 shadow">
          <h2 className="text-xl font-semibold mb-4">Simulator & History</h2>

          <div className="flex items-center gap-3">
            <label className="flex items-center gap-2">
              <div className="text-sm">Matches to simulate</div>
              <input type="number" min={1} max={20} value={simMatches} onChange={(e) => setSimMatches(Math.max(1, Math.min(20, Number(e.target.value))))} className="w-20 ml-2 p-2 rounded-md border" />
            </label>

            <button onClick={runSimulation} className="ml-4 px-4 py-2 rounded-lg bg-green-600 text-white">Run Simulation</button>
            <button onClick={resetHistory} className="px-4 py-2 rounded-lg border">Clear</button>
          </div>

          {history.length === 0 ? (
            <div className="mt-4 text-sm text-gray-600">No simulation history yet — run a simulation to see example match outcomes.</div>
          ) : (
            <div className="mt-4 overflow-x-auto">
              <table className="min-w-full text-left">
                <thead>
                  <tr className="text-xs uppercase text-gray-500">
                    <th className="px-3 py-2">#</th>
                    <th className="px-3 py-2">Kills</th>
                    <th className="px-3 py-2">Deaths</th>
                    <th className="px-3 py-2">Assists</th>
                    <th className="px-3 py-2">KDA</th>
                  </tr>
                </thead>
                <tbody>
                  {history.map((h, i) => (
                    <tr key={i} className="border-t">
                      <td className="px-3 py-2">{i + 1}</td>
                      <td className="px-3 py-2">{h.k}</td>
                      <td className="px-3 py-2">{h.d}</td>
                      <td className="px-3 py-2">{h.a}</td>
                      <td className="px-3 py-2">{formatNum(h.kda)}</td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          )}
        </section>

        <section className="md:col-span-2 bg-white rounded-2xl p-6 shadow">
          <h2 className="text-xl font-semibold mb-4">Utilities</h2>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-3">
            <div className="p-3 rounded-lg border">
              <div className="text-sm text-gray-600">Export current inputs</div>
              <pre className="text-xs bg-gray-50 p-2 mt-2 rounded">{`{ kills: ${kills}, deaths: ${deaths}, assists: ${assists}, kda: ${formatNum(currentKda)} }`}</pre>
            </div>

            <div className="p-3 rounded-lg border">
              <div className="text-sm text-gray-600">Tip</div>
              <div className="mt-2 text-sm">To avoid divide-by-zero issues many players treat deaths as at least 1 for KDA calculations. Use consistent definition when comparing.</div>
            </div>

            <div className="p-3 rounded-lg border">
              <div className="text-sm text-gray-600">Quick formula</div>
              <div className="mt-2 text-sm">KDA ≈ (Kills + Assists) / max(1, Deaths)</div>
            </div>
          </div>
        </section>

        <footer className="md:col-span-2 text-center text-sm text-gray-500 mt-4">
          Built with ❤️ — tweak the source to add features like match-by-match upload, weighted averages, or a backend to store history.
        </footer>
      </main>
    </div>
  );
}
