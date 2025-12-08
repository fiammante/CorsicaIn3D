# Horizon Distance Calculator with Atmospheric Refraction

## Physical Background

### Earth's Curvature and Line of Sight

The distance to the horizon is fundamentally limited by Earth's curvature. When standing at a given height, light travels in a straight line (in vacuum) from your eye to the horizon point where the line becomes tangent to Earth's surface. Beyond this point, the surface curves away below your line of sight.

### Atmospheric Refraction

Atmospheric refraction is the bending of light rays as they pass through layers of air with different densities. This phenomenon significantly affects horizon distance calculations.

#### Physical Mechanism

1. **Density Gradient**: Earth's atmosphere decreases in density with altitude. Near the surface, air is denser; at higher altitudes, it becomes progressively thinner.

2. **Refractive Index**: The refractive index of air (n) depends on its density, which varies with temperature, pressure, and humidity. At sea level under standard conditions, n ≈ 1.000293 (very close to 1, but measurable).

3. **Light Path Bending**: According to Snell's law, when light passes from one medium to another with different refractive indices, it bends. In the atmosphere, light traveling horizontally or at small angles continuously passes through layers of decreasing density, causing the ray path to curve downward.

4. **Curved Ray Paths**: Because atmospheric density decreases gradually (not abruptly), light rays follow a curved path rather than bending at discrete interfaces. The ray curves downward, following Earth's curvature more closely than it would in vacuum.

#### Mathematical Effect

The curvature of light rays means that:
- Light can "see around" Earth's curvature to some extent
- The visible horizon extends beyond what simple geometry would predict
- The effect is equivalent to Earth having a larger apparent radius

**Standard Atmospheric Model**: Under typical atmospheric conditions, refraction makes Earth appear to have a radius approximately **1.2 times** its actual radius:

```
R_apparent = k × R_actual
k ≈ 7.68 / 6.371 ≈ 1.205
```

Where:
- R_actual = 6,371 km (mean Earth radius)
- R_apparent = 7,680 km (effective radius with refraction)
- k ≈ 1.2 (refraction coefficient)

#### Variability of Refraction

Atmospheric refraction is **not constant** and varies with:
- **Temperature gradients**: Strong temperature inversions can create mirages or superior mirages
- **Pressure**: Higher pressure increases air density and refraction
- **Humidity**: Water vapor affects refractive index
- **Time of day**: Temperature profiles change between day and night
- **Weather conditions**: Stable vs. turbulent air

The value k = 1.2 is a **statistical average** for standard atmospheric conditions. In extreme cases:
- Cold air over warm water: k can approach 1.0 (minimal refraction)
- Strong inversions: k can exceed 1.3-1.4 (extreme refraction, mirages possible)

## Mathematical Formulas

### Direct Formula (Distance to Horizon)

```
c = f × (√a + √b)
```

Where:
- **c**: Distance to horizon or object (km)
- **a**: View height - observer's eye height above ground (m)
- **b**: Object height above ground (m)
- **f**: Geometric factor = √(2R), unit is 1000×√m

### Factor Calculation

```
f = √(2R)
```

Where **R** is Earth's radius in thousands of kilometers:
- **Without refraction**: R = 6.371 (actual Earth radius)
- **With refraction**: R = 7.68 (apparent Earth radius)

This gives:
- **With refraction**: f ≈ 3.923
- **Without refraction**: f ≈ 3.572

### Inverted Formula (Minimum Object Height)

Solving for **b** from the direct formula:

```
c/f = √a + √b
√b = c/f - √a
b = (c/f - √a)²
```

## Python Implementation

```python
import math

def calculate_horizon_distance(view_height_m, object_height_m=0, 
                                atmospheric_refraction=True):
    """
    Calculate distance to horizon based on view height and optional object height.
    
    Parameters:
    -----------
    view_height_m : float
        Height of observer's eyes above ground (meters)
    object_height_m : float, optional
        Height of distant object above ground (meters), default 0
    atmospheric_refraction : bool, optional
        Include atmospheric refraction effect (default True)
    
    Returns:
    --------
    float
        Distance to horizon or object in kilometers
    
    Formula:
    --------
    c = f * (√a + √b)
    f = √(2R)
    
    Where:
    - R = 6.371 (actual Earth radius in 1000 km) without refraction
    - R = 7.68 (apparent Earth radius in 1000 km) with refraction
    """
    # Earth radius in 1000 km
    if atmospheric_refraction:
        R = 7.68  # Mean apparent Earth radius
    else:
        R = 6.371  # Mean actual Earth radius
    
    # Calculate factor f = √(2R), unit is 1000 * √m
    f = math.sqrt(2 * R)
    
    # Calculate distance in km: c = f * (√a + √b)
    distance_km = f * (math.sqrt(view_height_m) + math.sqrt(object_height_m))
    
    return distance_km


def calculate_minimum_object_height(view_height_m, distance_km, 
                                    atmospheric_refraction=True):
    """
    Calculate minimum object height to be visible at given distance.
    
    Inverted formula from: c = f * (√a + √b)
    Solving for b: b = (c/f - √a)²
    
    Parameters:
    -----------
    view_height_m : float
        Height of observer's eyes above ground (meters)
    distance_km : float
        Distance to object (kilometers)
    atmospheric_refraction : bool, optional
        Include atmospheric refraction effect (default True)
    
    Returns:
    --------
    float
        Minimum object height in meters, or None if object is beyond horizon
    """
    # Earth radius in 1000 km
    if atmospheric_refraction:
        R = 7.68  # Mean apparent Earth radius
    else:
        R = 6.371  # Mean actual Earth radius
    
    f = math.sqrt(2 * R)
    
    # Inverted formula: b = (c/f - √a)²
    sqrt_view_height = math.sqrt(view_height_m)
    term = distance_km / f - sqrt_view_height
    
    if term < 0:
        # Object is beyond horizon even at ground level
        return None
    
    object_height_m = term ** 2
    
    return object_height_m


def compare_refraction_effects(view_height_m, distances_km):
    """
    Compare horizon calculations with and without atmospheric refraction.
    
    Parameters:
    -----------
    view_height_m : float
        Height of observer's eyes above ground (meters)
    distances_km : list of float
        List of distances to evaluate (kilometers)
    """
    print(f"\n{'='*70}")
    print(f"Atmospheric Refraction Comparison")
    print(f"View Height: {view_height_m} m")
    print(f"{'='*70}\n")
    
    print(f"{'Distance (km)':<15} {'With Refraction (m)':<25} {'Without Refraction (m)':<25} {'Difference (m)':<15}")
    print(f"{'-'*15} {'-'*25} {'-'*25} {'-'*15}")
    
    for distance in distances_km:
        with_refr = calculate_minimum_object_height(view_height_m, distance, 
                                                     atmospheric_refraction=True)
        without_refr = calculate_minimum_object_height(view_height_m, distance, 
                                                       atmospheric_refraction=False)
        
        if with_refr is not None and without_refr is not None:
            difference = without_refr - with_refr
            print(f"{distance:<15.1f} {with_refr:<25.2f} {without_refr:<25.2f} {difference:<15.2f}")
        else:
            print(f"{distance:<15.1f} {'Beyond horizon':<25} {'Beyond horizon':<25} {'-':<15}")


# Example usage
if __name__ == "__main__":
    print("="*70)
    print("HORIZON DISTANCE CALCULATOR")
    print("="*70)
    
    # Example 1: Standard horizon distance from different heights
    print("\n1. Horizon Distance from Various Heights (with refraction)")
    print("-" * 70)
    heights = [1.70, 10, 50, 100, 500, 1000]
    
    for height in heights:
        distance = calculate_horizon_distance(height)
        print(f"Height: {height:>6.2f} m  →  Horizon distance: {distance:>6.2f} km")
    
    # Example 2: Main calculation - minimum object height
    print("\n" + "="*70)
    print("2. Minimum Visible Object Height Calculation")
    print("="*70)
    
    view_height = 50  # meters
    distance = 200  # kilometers
    
    min_height = calculate_minimum_object_height(view_height, distance)
    
    if min_height is not None:
        print(f"\nInitial conditions:")
        print(f"  View height: {view_height} m")
        print(f"  Distance: {distance} km")
        print(f"\nResult:")
        print(f"  Minimum visible object height: {min_height:.2f} m")
        
        # Verify by computing distance with this height
        computed_distance = calculate_horizon_distance(view_height, min_height)
        print(f"\nVerification:")
        print(f"  Distance computed with result height: {computed_distance:.2f} km")
        print(f"  Error: {abs(computed_distance - distance):.6f} km")
        
        # Also show without atmospheric refraction
        min_height_no_refr = calculate_minimum_object_height(view_height, distance, 
                                                             atmospheric_refraction=False)
        print(f"\nWithout atmospheric refraction:")
        print(f"  Minimum visible object height: {min_height_no_refr:.2f} m")
        print(f"  Difference due to refraction: {min_height_no_refr - min_height:.2f} m")
    else:
        print(f"\nObject at {distance} km is beyond horizon from {view_height} m height")
    
    # Example 3: Comparison table
    print("\n" + "="*70)
    print("3. Refraction Effect at Different Distances")
    print("="*70)
    
    compare_refraction_effects(50, [50, 100, 150, 200, 250, 300])
    
    # Example 4: Real-world scenarios
    print("\n" + "="*70)
    print("4. Real-World Scenarios")
    print("="*70)
    
    scenarios = [
        ("Person on beach (eye level)", 1.70, 0),
        ("Lighthouse keeper (30m)", 30, 0),
        ("Aircraft at 10,000m", 10000, 0),
        ("Viewing Mt. Everest (8849m) from 100m", 100, 8849),
    ]
    
    print(f"\n{'Scenario':<40} {'Distance (km)':<15}")
    print(f"{'-'*40} {'-'*15}")
    
    for scenario, view_h, obj_h in scenarios:
        dist = calculate_horizon_distance(view_h, obj_h)
        print(f"{scenario:<40} {dist:<15.2f}")
```

## Physical Insights

### Why Square Root Dependency?

The formula uses √h (square root of height) rather than h directly. This comes from the Pythagorean theorem applied to Earth's spherical geometry:

For a tangent line from height h to Earth's surface of radius R:
```
(R + h)² = R² + d²
R² + 2Rh + h² = R² + d²
d² = 2Rh + h²
```

For h << R (height much smaller than Earth's radius):
```
d² ≈ 2Rh
d ≈ √(2Rh)
```

This is why distance scales with √h, not linearly with h.

### Refraction Impact

The ~9.8% increase in visible distance due to refraction (from R = 6.371 to R = 7.68) means:
- At 50m height: horizon extends from ~25.3 km to ~27.8 km (~2.5 km extra)
- At 200 km distance: object can be ~9.8% shorter and still visible

### Practical Applications

1. **Maritime navigation**: Ships appearing/disappearing over horizon
2. **Radio communications**: Line-of-sight radio links
3. **Surveying**: Visibility calculations for towers, landmarks
4. **Aviation**: Determining when ground features become visible
5. **Astronomy**: Calculating refraction effects near horizon

## References

- Earth's mean radius: 6,371 km (IUGG)
- Standard atmospheric refraction coefficient: k ≈ 1.2
- Formula source: [rechneronline.de](https://rechneronline.de/sehwinkel/distance-horizon.php)

## Notes

- Formulas assume spherical Earth (good approximation for these distances)
- Atmospheric refraction value is average; actual conditions vary
- Does not account for obstacles, terrain, or visual acuity limits
- For very long distances (>500 km), Earth's ellipsoidal shape becomes relevant
