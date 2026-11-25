# GRLINS MEOR Dataset

Use this dataset together with the GRLINS paper to reproduce selected multi-epoch outlier rejection (MEOR) experiments in Table 3. The dataset covers four error cases on the `playground00` sequence with random seed `0` and a target outlier injection ratio of 75%.

## Files

| File | Contents |
| --- | --- |
| `integer_cycle_slip_seed0_ratio75.csv` | Observations and labels for integer-cycle slips |
| `half_cycle_slip_seed0_ratio75.csv` | Observations and labels for half-cycle slips |
| `quarter_cycle_slip_seed0_ratio75.csv` | Observations and labels for quarter-cycle slips |
| `temporally_correlated_error_seed0_ratio75.csv` | Observations and labels for temporally correlated errors |
| `trajectory.tum` | Antenna trajectory in the world frame, shared by all four cases |

## Data Format

The **CSV files** have a header row and one rover–base time-differenced carrier-phase (TDCP) observation per data row.

| Field | Unit | Definition |
| --- | --- | --- |
| `t_prev_s`, `t_curr_s` | s | GPS seconds of week for the previous and current GNSS epochs |
| `signal_id` | — | Anonymous satellite signal ID corresponding to the superscript $s$ in the paper. The same IDs are used in all four cases. |
| `tdcp_m` | m | $\lambda_s\Delta\tilde\varphi^s_{j-1,j}$ in Eqs. (20) and (30). Subtract the base-station observation from the rover observation, then subtract the previous-epoch value from the current-epoch value. |
| `sigma_tdcp_m` | m | Standard deviation $\lambda_s\sigma^s_{\Delta\varphi,j-1,j}$ in Eq. (31) |
| `los_x`, `los_y`, `los_z` | — | Components of the line-of-sight (LOS) vector $\mathbf e^s_{r,j}$ in Eq. (30). At the current epoch, the vector points from the satellite to the rover antenna and is expressed in the world frame. |
| `geom_corr_m` | m | Combined geometric term $\mathcal T_{\mathrm{LOS}}-\mathcal T_{\mathrm{SAT}}-\rho^s_{m,j-1,j}$ in Eq. (30) |
| `dd_outlier_label` | — | Reference label derived from the triple-difference method presented in the GRLINS paper. `1` indicates a healthy observation and `0` indicates an outlier. |

The **TUM file** is space-separated with columns `timestamp tx ty tz qx qy qz qw`. Timestamps are in GPS seconds of week, antenna positions are in meters in the world frame, and quaternion components are ordered as `xyzw`.

## How to Use the Data

1. Group the TDCP observations into multi-epoch windows using their timestamps.
2. Use the TUM trajectory to compute the relative displacement term $\mathcal T_{\mathrm{REL}}$ in Eq. (29).
3. Form the normalized residuals in Eq. (31) using the TDCP observations, LOS vectors, and combined geometric terms from the CSV, together with the initial-epoch attitude, relative displacements, and receiver clock-bias changes between adjacent epochs.
4. Follow Eqs. (31)–(36) to apply graduated non-convexity with the Geman–McClure loss (GNC-GM). Initialize the attitude at the first epoch of each window from the TUM quaternion. Keep roll and pitch fixed and use the yaw from the quaternion as the initial estimate. Estimate one receiver clock-bias change per interval, initialized to zero.
5. Compute the TDCP residuals after optimization. Classify observations using the `0.02 m` threshold and compare the results with `dd_outlier_label`.

## License

This dataset is licensed under the
[Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

See [LICENSE](LICENSE) for details.