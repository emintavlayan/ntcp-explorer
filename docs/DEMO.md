# Demonstrating NTCP Explorer

**Work in progress — NTCP Explorer research prototype.** Model coverage, interfaces and documentation may change; clinical validation is pending.

## Run locally

From the repository root:

```powershell
git pull --ff-only
dotnet run
```

Open <http://localhost:8080>. The application loads a synthetic example automatically.

## A short demonstration

1. Show the heart, lung and liver mean-dose inputs, selectable dose-response curves, plan markers and endpoint table.
2. Change mean lung dose from 14 Gy to 20 Gy. Inspect the marker and endpoint estimate returned by the server.
3. Compare the adaptive plot with **Full range**; the latter restores the fixed mathematical display range.
4. Open **NTCP models** to inspect source-linked equations, coefficients, units, required inputs and implementation status across the catalogue.
5. Evaluate an implemented model in the catalogue runner with its required inputs, then return to **Calculator** to confirm the calculator inputs are preserved.

The main calculator's three equations use dose only. Additional catalogue models require their own explicit clinical and dosimetric inputs. Relative-risk and odds-ratio outputs retain their original scales.

The outputs concern different endpoints and follow-up periods. They are not summed, and their percentages are not presented as equivalent measures of clinical severity.

## Included models

| Organ | Endpoint | Equation | Parameter source |
| --- | --- | --- | --- |
| Heart | Cardiac mortality | Linear: `a × D` | Brodin 2014, Table 1, derived from Hancock 1993; `a = 0.00105 Gy⁻¹` |
| Lungs | Radiation pneumonitis | Logistic: `1 / (1 + exp(−(a + b × D)))` | Marks / QUANTEC 2010; `a = −3.87`, `b = 0.126 Gy⁻¹` |
| Liver | Radiation-induced liver disease | LKB with `n = 1`: `Φ((D − TD50) / (m × TD50))` | Brodin 2014, Table 1, citing Pan / QUANTEC 2010; `TD50 = 39.8 Gy`, `m = 0.12` |

Here `D` is the required mean organ dose in physical Gy. The plotted 0–50 Gy range is a display range, not a statement of model validation across that range. The equations are for conventionally fractionated source settings; no automatic fractionation conversion is performed.

## Run a server bundle

```powershell
dotnet run -- Bundle
cd deploy
$env:ASPNETCORE_URLS = 'http://0.0.0.0:8080'
dotnet Server.dll
```

The bundle contains the server and compiled client assets. The host requires the .NET 8 ASP.NET Core runtime. Use the host's HTTPS reverse proxy when exposing the application outside a local demonstration.

Alternatively, build the supplied Docker image:

```sh
docker build -t ntcp-explorer .
docker run --rm -p 8080:8080 ntcp-explorer
```

## Verification

```powershell
dotnet run -- RunTestsHeadless
```

The server checks published-equation reference values, the normal CDF, curve/marker consistency, monotonicity, unsupported clinical-factor effects, and input validation. Client tests check stale responses, catalogue navigation, empty numeric fields, and resetting the example.

This is a research demonstration and has not been validated for clinical decisions.
