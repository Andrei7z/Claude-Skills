# LCF5 Energy Data Skill

## Overview
LCF5 is a glasshouse facility with CHP engines, boilers, heat pumps, and CO2 systems. Energy data is stored at 15-minute intervals.

## Primary Data Source
Use **only** `dbo.qryEnergiData_15Min` for all LCF5 queries.

## Key Column: DateTimeLinkEnergy
- Type: `smalldatetime`
- Primary key / timestamp for all records
- 15-minute intervals
- Helper columns: `Date`, `WeekComm` (week commencing), `YearMonth` (format: YYYY-MM)

## Column Naming Convention
Most columns follow the pattern: `[Type]_[Asset]_[Agg|Diff]`
- **_Agg** = Cumulative/aggregate meter reading
- **_Diff** = Interval consumption (use this for energy analysis)

## Equipment Categories

### CHP Engines (3 units: CHP1, CHP2, CHP3)
| Prefix | Meaning | Unit |
|--------|---------|------|
| Gas_CHPx | Gas consumption | kWh |
| Gen_CHPx | Electricity generated | kWh |
| Heat_CHPx | Heat output | kWh |
| CO2_CHPx | CO2 captured/produced | kg |

### Boilers (2 units: Boiler1, Boiler2)
| Prefix | Meaning | Unit |
|--------|---------|------|
| Gas_Boilerx | Gas consumption | kWh |
| Heat_Boilerx | Heat output | kWh |
| Hrs_Boilerx | Run hours | hours |

### Heat Pumps (16 units: HP1–HP16)
| Prefix | Meaning | Unit |
|--------|---------|------|
| Consum_HPx | Electricity consumption | kWh |

### Grid Electricity
| Column | Meaning |
|--------|---------|
| Imp_EnergyCentre | Import from grid |
| Exp_EnergyCentre | Export to grid |
| Energy_Tariff2_Reverse | Secondary tariff meter |

### Glasshouse Electricity
| Column | Meaning |
|--------|---------|
| Elec_GHN | Glasshouse North consumption |
| Elec_GHS | Glasshouse South consumption |

### Heat Distribution
| Column | Meaning |
|--------|---------|
| Heat_Total | Total heat to site |
| Heat_RHI | RHI-eligible heat |
| Heat_Pump | Heat pump output |
| Heat_CO2 | Heat for CO2 system |
| Heat_DAC | Direct air capture heat |
| Heat_Effluent | Effluent heat recovery |
| Heat_2_1 | Inter-zone heat transfer |

### CO2 System
| Column | Meaning |
|--------|---------|
| CO2_Chrg_Grow_KG | CO2 charged to growing (kg) |
| CO2_Chrg_Grow_HRS | CO2 charging run hours |

### Temperatures (point-in-time, not _Diff)
| Prefix | Meaning |
|--------|---------|
| Temp_HT01–HT10 | High-temperature sensors |
| Temp_LT01–LT10 | Low-temperature sensors |
| Temp_LTIn / Temp_LTOut | Low-temp circuit in/out |
| Temp_GH | Glasshouse temperature |
| Temp_Ambient_OS | Outside ambient temperature |
| SetpP_GHN / SetpP_GHS | Glasshouse setpoints (North/South) |

### Status Flags (bit columns)
| Column | Meaning |
|--------|---------|
| Daylight | Daylight hours flag |
| UKPN_Zero_Export_Mode | Grid export restriction active |
| Co2_Lead | CO2 system leading |
| Thermal_Lead | Thermal demand leading |
| Facility_Lead | Facility demand leading |

## Common Query Patterns

### Daily totals
```sql
SELECT Date, 
       SUM(Gen_CHP1_Diff + Gen_CHP2_Diff + Gen_CHP3_Diff) AS Total_CHP_Gen,
       SUM(Imp_EnergyCentre_Diff) AS Grid_Import
FROM dbo.qryEnergiData_15Min
WHERE Date >= '2024-01-01'
GROUP BY Date
ORDER BY Date
```

### Weekly aggregation
```sql
SELECT WeekComm,
       SUM(Heat_Total_Diff) AS Weekly_Heat
FROM dbo.qryEnergiData_15Min
GROUP BY WeekComm
ORDER BY WeekComm
```

### CHP efficiency check
```sql
SELECT Date,
       SUM(Gas_CHP1_Diff) AS Gas_In,
       SUM(Gen_CHP1_Diff) AS Elec_Out,
       SUM(Heat_CHP1_Diff) AS Heat_Out,
       (SUM(Gen_CHP1_Diff) + SUM(Heat_CHP1_Diff)) / NULLIF(SUM(Gas_CHP1_Diff), 0) * 100 AS Efficiency_Pct
FROM dbo.qryEnergiData_15Min
WHERE Date >= '2024-01-01'
GROUP BY Date
```

## Notes
- Always use `_Diff` columns for consumption/generation analysis
- Use `_Agg` columns only when you need cumulative readings
- Temperature columns are instantaneous readings (no _Diff)
- Bit flags are status indicators, not energy values
