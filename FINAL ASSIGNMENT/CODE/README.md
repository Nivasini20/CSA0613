import csv
import os
import time
import random

# ==========================================================
# INTELLIGENT LOGISTICS RESOURCE OPTIMIZATION
# CSA0613 - DESIGN AND ANALYSIS OF ALGORITHMS
# ==========================================================

DATASET = "delivery_logistics.csv"
RESULT_FILE = "final_results.csv"


# ==========================================================
# 1. CREATE DATASET IF IT DOES NOT EXIST
# ==========================================================

def create_dataset():

    print("Dataset not found.")
    print("Creating 10,000-package logistics dataset...\n")

    random.seed(42)

    with open(DATASET, "w", newline="") as file:

        writer = csv.writer(file)

        writer.writerow([
            "Package_ID",
            "Weight",
            "Volume",
            "Value",
            "Priority",
            "Deadline"
        ])

        for i in range(1, 10001):

            weight = round(random.uniform(0.5, 50), 2)

            volume = round(
                weight / random.uniform(180, 320),
                3
            )

            value = round(
                weight * random.uniform(70, 180)
                + random.uniform(100, 1500),
                2
            )

            priority = random.choice([
                "Normal",
                "High",
                "Urgent"
            ])

            deadline = random.randint(6, 72)

            writer.writerow([
                "P" + str(i),
                weight,
                volume,
                value,
                priority,
                deadline
            ])

    print("Dataset created successfully!\n")


# ==========================================================
# 2. LOAD DATASET
# ==========================================================

def load_dataset():

    packages = []

    with open(DATASET, "r") as file:

        reader = csv.DictReader(file)

        for row in reader:

            package = {
                "id": row["Package_ID"],
                "weight": float(row["Weight"]),
                "volume": float(row["Volume"]),
                "value": float(row["Value"]),
                "priority": row["Priority"],
                "deadline": int(row["Deadline"])
            }

            packages.append(package)

    return packages


# ==========================================================
# 3. GREEDY ALGORITHM
# ==========================================================

def greedy_algorithm(packages, capacity):

    start = time.perf_counter()

    # Calculate value / weight ratio
    sorted_packages = sorted(
        packages,
        key=lambda p: p["value"] / p["weight"],
        reverse=True
    )

    selected = []

    total_weight = 0
    total_volume = 0
    total_value = 0

    for package in sorted_packages:

        if total_weight + package["weight"] <= capacity:

            selected.append(package)

            total_weight += package["weight"]
            total_volume += package["volume"]
            total_value += package["value"]

    execution_time = time.perf_counter() - start

    return (
        selected,
        total_value,
        total_weight,
        total_volume,
        execution_time
    )


# ==========================================================
# 4. DYNAMIC PROGRAMMING
# ==========================================================

def dynamic_programming(packages, capacity):

    start = time.perf_counter()

    # Convert weight to integer
    scale = 10

    weights = [
        max(1, int(p["weight"] * scale))
        for p in packages
    ]

    values = [
        p["value"]
        for p in packages
    ]

    C = int(capacity * scale)

    # DP table
    dp = [0] * (C + 1)

    # Store selected package indexes
    selected_items = [[] for _ in range(C + 1)]

    for i in range(len(packages)):

        weight = weights[i]
        value = values[i]

        for c in range(C, weight - 1, -1):

            new_value = dp[c - weight] + value

            if new_value > dp[c]:

                dp[c] = new_value

                selected_items[c] = (
                    selected_items[c - weight] + [i]
                )

    best_capacity = max(
        range(C + 1),
        key=lambda x: dp[x]
    )

    indexes = selected_items[best_capacity]

    selected = [
        packages[i]
        for i in indexes
    ]

    total_weight = sum(
        p["weight"] for p in selected
    )

    total_volume = sum(
        p["volume"] for p in selected
    )

    total_value = sum(
        p["value"] for p in selected
    )

    execution_time = time.perf_counter() - start

    return (
        selected,
        total_value,
        total_weight,
        total_volume,
        execution_time
    )


# ==========================================================
# 5. PROPOSED HYBRID ALGORITHM
# ==========================================================

def hybrid_algorithm(packages, capacity):

    start = time.perf_counter()

    # ------------------------------------------------------
    # STEP 1: Calculate intelligent score
    # ------------------------------------------------------

    def score(package):

        ratio = package["value"] / package["weight"]

        priority_bonus = {
            "Normal": 1.0,
            "High": 1.10,
            "Urgent": 1.20
        }

        deadline_bonus = 1.0

        if package["deadline"] <= 12:
            deadline_bonus = 1.10

        return (
            ratio
            * priority_bonus[package["priority"]]
            * deadline_bonus
        )

    # ------------------------------------------------------
    # STEP 2: Rank packages
    # ------------------------------------------------------

    ranked = sorted(
        packages,
        key=score,
        reverse=True
    )

    # ------------------------------------------------------
    # STEP 3: Select promising packages
    # ------------------------------------------------------

    shortlist_size = min(
        300,
        len(ranked)
    )

    shortlist = ranked[:shortlist_size]

    # ------------------------------------------------------
    # STEP 4: Apply Greedy to shortlist
    # ------------------------------------------------------

    selected = []

    total_weight = 0
    total_volume = 0
    total_value = 0

    for package in shortlist:

        if total_weight + package["weight"] <= capacity:

            selected.append(package)

            total_weight += package["weight"]
            total_volume += package["volume"]
            total_value += package["value"]

    # ------------------------------------------------------
    # STEP 5: Local improvement
    # ------------------------------------------------------

    selected_ids = set(
        p["id"] for p in selected
    )

    remaining = ranked[shortlist_size:]

    for package in remaining:

        if package["id"] not in selected_ids:

            if total_weight + package["weight"] <= capacity:

                selected.append(package)

                selected_ids.add(package["id"])

                total_weight += package["weight"]
                total_volume += package["volume"]
                total_value += package["value"]

    execution_time = time.perf_counter() - start

    return (
        selected,
        total_value,
        total_weight,
        total_volume,
        execution_time
    )


# ==========================================================
# 6. DISPLAY RESULT
# ==========================================================

def display_result(
        algorithm,
        selected,
        value,
        weight,
        volume,
        capacity,
        time_taken):

    utilization = (
        weight / capacity
    ) * 100

    print("\nAlgorithm:", algorithm)

    print(
        "Selected Packages :",
        len(selected)
    )

    print(
        "Total Value        :",
        round(value, 2)
    )

    print(
        "Weight Used        :",
        round(weight, 2),
        "kg"
    )

    print(
        "Volume Used        :",
        round(volume, 3),
        "m3"
    )

    print(
        "Capacity Utilized  :",
        round(utilization, 2),
        "%"
    )

    print(
        "Execution Time     :",
        round(time_taken, 6),
        "seconds"
    )


# ==========================================================
# 7. RUN EXPERIMENTS
# ==========================================================

def run_experiments(packages):

    package_sizes = [
        100,
        500,
        1000,
        5000,
        10000
    ]

    capacities = [
        500,
        1000,
        2500,
        5000
    ]

    results = []

    print("\n")
    print("=" * 90)
    print("        LOGISTICS RESOURCE OPTIMIZATION EXPERIMENT")
    print("=" * 90)

    for size in package_sizes:

        subset = packages[:size]

        for capacity in capacities:

            print("\n")
            print("-" * 90)

            print(
                "Packages:",
                size,
                "| Vehicle Capacity:",
                capacity,
                "kg"
            )

            # ------------------------------------------------
            # GREEDY
            # ------------------------------------------------

            (
                greedy_selected,
                greedy_value,
                greedy_weight,
                greedy_volume,
                greedy_time
            ) = greedy_algorithm(
                subset,
                capacity
            )

            # ------------------------------------------------
            # DYNAMIC PROGRAMMING
            # ------------------------------------------------

            # DP becomes expensive for very large cases.
            # Run DP up to 1000 packages.
            if size <= 1000:

                (
                    dp_selected,
                    dp_value,
                    dp_weight,
                    dp_volume,
                    dp_time
                ) = dynamic_programming(
                    subset,
                    capacity
                )

            else:

                dp_selected = []
                dp_value = 0
                dp_weight = 0
                dp_volume = 0
                dp_time = 0

            # ------------------------------------------------
            # HYBRID
            # ------------------------------------------------

            (
                hybrid_selected,
                hybrid_value,
                hybrid_weight,
                hybrid_volume,
                hybrid_time
            ) = hybrid_algorithm(
                subset,
                capacity
            )

            # ------------------------------------------------
            # DISPLAY
            # ------------------------------------------------

            display_result(
                "GREEDY",
                greedy_selected,
                greedy_value,
                greedy_weight,
                greedy_volume,
                capacity,
                greedy_time
            )

            if size <= 1000:

                display_result(
                    "DYNAMIC PROGRAMMING",
                    dp_selected,
                    dp_value,
                    dp_weight,
                    dp_volume,
                    capacity,
                    dp_time
                )

            else:

                print("\nAlgorithm: DYNAMIC PROGRAMMING")
                print("Skipped for large dataset to demonstrate scalability limitation.")

            display_result(
                "PROPOSED HYBRID",
                hybrid_selected,
                hybrid_value,
                hybrid_weight,
                hybrid_volume,
                capacity,
                hybrid_time
            )

            # ------------------------------------------------
            # SAVE RESULTS
            # ------------------------------------------------

            results.append({

                "Packages": size,

                "Capacity": capacity,

                "Greedy_Value":
                    round(greedy_value, 2),

                "DP_Value":
                    round(dp_value, 2),

                "Hybrid_Value":
                    round(hybrid_value, 2),

                "Greedy_Selected":
                    len(greedy_selected),

                "DP_Selected":
                    len(dp_selected),

                "Hybrid_Selected":
                    len(hybrid_selected),

                "Greedy_Weight":
                    round(greedy_weight, 2),

                "DP_Weight":
                    round(dp_weight, 2),

                "Hybrid_Weight":
                    round(hybrid_weight, 2),

                "Greedy_Utilization":
                    round(
                        greedy_weight /
                        capacity * 100,
                        2
                    ),

                "DP_Utilization":
                    round(
                        dp_weight /
                        capacity * 100,
                        2
                    ),

                "Hybrid_Utilization":
                    round(
                        hybrid_weight /
                        capacity * 100,
                        2
                    ),

                "Greedy_Time":
                    round(greedy_time, 6),

                "DP_Time":
                    round(dp_time, 6),

                "Hybrid_Time":
                    round(hybrid_time, 6)
            })

    return results


# ==========================================================
# 8. SAVE RESULTS TO CSV
# ==========================================================

def save_results(results):

    if not results:
        return

    with open(
        RESULT_FILE,
        "w",
        newline=""
    ) as file:

        writer = csv.DictWriter(
            file,
            fieldnames=results[0].keys()
        )

        writer.writeheader()

        writer.writerows(results)

    print("\n")
    print("=" * 90)
    print("RESULT FILE CREATED")
    print("=" * 90)

    print(
        "Saved as:",
        RESULT_FILE
    )


# ==========================================================
# 9. FINAL COMPARISON
# ==========================================================

def final_comparison(results):

    print("\n")
    print("=" * 90)
    print("                 FINAL ALGORITHM COMPARISON")
    print("=" * 90)

    print(
        f"{'Packages':<10}"
        f"{'Capacity':<12}"
        f"{'Greedy':<15}"
        f"{'DP':<15}"
        f"{'Hybrid':<15}"
    )

    print("-" * 90)

    for row in results:

        print(
            f"{row['Packages']:<10}"
            f"{row['Capacity']:<12}"
            f"{row['Greedy_Value']:<15}"
            f"{row['DP_Value']:<15}"
            f"{row['Hybrid_Value']:<15}"
        )


# ==========================================================
# 10. MAIN PROGRAM
# ==========================================================

def main():

    print("\n")
    print("=" * 90)
    print("       INTELLIGENT LOGISTICS RESOURCE OPTIMIZATION")
    print("=" * 90)

    print("\nCSA0613 - Design and Analysis of Algorithms")

    # Create dataset if necessary
    if not os.path.exists(DATASET):
        create_dataset()

    # Load dataset
    packages = load_dataset()

    print("\nDataset loaded successfully!")

    print(
        "Total Packages:",
        len(packages)
    )

    print("\nFirst 5 Packages:")

    for package in packages[:5]:

        print(
            package["id"],
            "| Weight:",
            package["weight"],
            "kg",
            "| Value:",
            package["value"],
            "| Priority:",
            package["priority"]
        )

    # Run experiments
    results = run_experiments(
        packages
    )

    # Save results
    save_results(
        results
    )

    # Final comparison
    final_comparison(
        results
    )

    print("\n")
    print("=" * 90)
    print("                 PROJECT COMPLETED")
    print("=" * 90)

    print("\nOutput files:")
    print("1. delivery_logistics.csv")
    print("2. final_results.csv")

    print("\nAll experiments completed successfully!")


# ==========================================================
# START PROGRAM
# ==========================================================

if __name__ == "__main__":
    main()
