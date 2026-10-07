"""
Resonant Frequency Calculator
-----------------------------
A menu-driven program for engineering students to calculate the resonant
frequency of LC and RLC circuits and related quantities.

Key relations:
    Resonant frequency     f0 = 1 / (2 * pi * sqrt(L * C))        (Hz)
    Angular frequency      w0 = 1 / sqrt(L * C)                   (rad/s)
    Inductance needed      L  = 1 / ((2 * pi * f0)^2 * C)
    Capacitance needed     C  = 1 / ((2 * pi * f0)^2 * L)

At resonance XL = XC, so the circuit behaves as a pure resistance.

Series RLC circuit:
    Quality factor         Q  = (1 / R) * sqrt(L / C)
    Bandwidth              BW = f0 / Q
    Half-power frequencies f1, f2 = f0 * (sqrt(1 + 1/(4Q^2)) -/+ 1/(2Q))
    Impedance at f0        Z = R (minimum)
    Voltage magnification  VL = VC = Q * V

Practical parallel circuit (coil with resistance R in parallel with C):
    Resonant frequency     f0 = (1 / (2 * pi)) * sqrt(1/(L*C) - (R/L)^2)
    Dynamic impedance      Zd = L / (C * R)   (maximum impedance)
"""

import math


# ---------------------------------------------------------------- input helpers
def get_positive_float(prompt):
    """Keep asking until the user enters a valid positive number."""
    while True:
        try:
            value = float(input(prompt))
            if value <= 0:
                print("  Please enter a value greater than zero.")
                continue
            return value
        except ValueError:
            print("  Invalid input. Please enter a number.")


def get_l(prompt="Inductance L (mH): "):
    """Ask for inductance in millihenry, return henry."""
    return get_positive_float(prompt) / 1e3


def get_c(prompt="Capacitance C (microfarad): "):
    """Ask for capacitance in microfarad, return farad."""
    return get_positive_float(prompt) / 1e6


# ------------------------------------------------------------------ core maths
def resonant_frequency(l, c):
    """f0 = 1 / (2 * pi * sqrt(L * C))"""
    return 1 / (2 * math.pi * math.sqrt(l * c))


def angular_frequency(l, c):
    """w0 = 1 / sqrt(L * C)"""
    return 1 / math.sqrt(l * c)


def find_inductance(f0, c):
    """L = 1 / ((2 * pi * f0)^2 * C)"""
    return 1 / ((2 * math.pi * f0) ** 2 * c)


def find_capacitance(f0, l):
    """C = 1 / ((2 * pi * f0)^2 * L)"""
    return 1 / ((2 * math.pi * f0) ** 2 * l)


def quality_factor_series(r, l, c):
    """Q = (1 / R) * sqrt(L / C)"""
    return math.sqrt(l / c) / r


def half_power_frequencies(f0, q):
    """Lower and upper half-power (cut-off) frequencies."""
    root = math.sqrt(1 + 1 / (4 * q ** 2))
    return f0 * (root - 1 / (2 * q)), f0 * (root + 1 / (2 * q))


def practical_parallel_f0(r, l, c):
    """Resonant frequency of a coil (L with series R) in parallel with C.
    Returns None when the term under the square root is negative."""
    term = 1 / (l * c) - (r / l) ** 2
    if term <= 0:
        return None
    return math.sqrt(term) / (2 * math.pi)


# ------------------------------------------------------------------- display
def format_frequency(f):
    """Show frequency in Hz, kHz or MHz."""
    if f >= 1e6:
        return f"{f / 1e6:.4f} MHz"
    if f >= 1e3:
        return f"{f / 1e3:.4f} kHz"
    return f"{f:.4f} Hz"


def lc_resonance():
    l = get_l()
    c = get_c()
    f0 = resonant_frequency(l, c)
    print("\n  ----- LC Resonance -----")
    print(f"  Resonant frequency f0 = {format_frequency(f0)}")
    print(f"  Angular frequency  w0 = {angular_frequency(l, c):.2f} rad/s")
    print(f"  Reactance at f0 XL=XC = {2 * math.pi * f0 * l:.3f} ohm")


def find_missing_component():
    f0 = get_positive_float("Required resonant frequency f0 (Hz): ")
    print("  Which component do you know?")
    print("   1. Inductance L")
    print("   2. Capacitance C")
    while True:
        pick = input("  Enter 1 or 2: ").strip()
        if pick in ("1", "2"):
            break
        print("  Please enter 1 or 2.")

    if pick == "1":
        l = get_l()
        c = find_capacitance(f0, l)
        print(f"\n  Capacitance required C = {c * 1e6:.4f} microfarad")
    else:
        c = get_c()
        l = find_inductance(f0, c)
        print(f"\n  Inductance required L = {l * 1e3:.4f} mH")


def series_rlc_resonance():
    r = get_positive_float("Resistance R (ohm): ")
    l = get_l()
    c = get_c()
    v = get_positive_float("Supply voltage V (V, rms): ")

    f0 = resonant_frequency(l, c)
    q = quality_factor_series(r, l, c)
    bw = f0 / q
    f1, f2 = half_power_frequencies(f0, q)
    i0 = v / r

    print("\n  ----- Series RLC Resonance -----")
    print(f"  Resonant frequency f0   = {format_frequency(f0)}")
    print(f"  Impedance at f0         = {r:.3f} ohm (minimum)")
    print(f"  Current at f0           = {i0:.4f} A (maximum)")
    print(f"  Quality factor Q        = {q:.3f}")
    print(f"  Bandwidth BW            = {format_frequency(bw)}")
    print(f"  Lower cut-off f1        = {format_frequency(f1)}")
    print(f"  Upper cut-off f2        = {format_frequency(f2)}")
    print(f"  Voltage across L and C  = {q * v:.2f} V each (Q x V)")
    sel = "highly selective" if q >= 10 else "moderately selective" if q >= 2 else "poorly selective"
    print(f"  Selectivity             = {sel}")


def parallel_resonance():
    r = get_positive_float("Coil resistance R (ohm): ")
    l = get_l()
    c = get_c()

    ideal = resonant_frequency(l, c)
    practical = practical_parallel_f0(r, l, c)

    print("\n  ----- Parallel Resonance -----")
    print(f"  Ideal f0 (R neglected)  = {format_frequency(ideal)}")
    if practical is None:
        print("  Coil resistance is too high: no resonance is possible.")
    else:
        print(f"  Practical f0            = {format_frequency(practical)}")
        print(f"  Dynamic impedance Zd    = {l / (c * r):,.2f} ohm (maximum)")


def menu():
    print("\n" + "=" * 52)
    print("        RESONANT FREQUENCY CALCULATOR")
    print("=" * 52)
    print(" 1. Resonant frequency of an LC circuit")
    print(" 2. Find L or C for a required frequency")
    print(" 3. Series RLC resonance (Q, bandwidth, f1, f2)")
    print(" 4. Practical parallel resonance (coil with R)")
    print(" 0. Exit")
    print("-" * 52)


def main():
    while True:
        menu()
        choice = input("Enter your choice: ").strip()

        if choice == "1":
            lc_resonance()
        elif choice == "2":
            find_missing_component()
        elif choice == "3":
            series_rlc_resonance()
        elif choice == "4":
            parallel_resonance()
        elif choice == "0":
            print("\nThank you for using the calculator. Goodbye!")
            break
        else:
            print("  Invalid choice. Please select from the menu.")


if __name__ == "__main__":
    main()
