import os
import statistics
from web3 import Web3

RPC_URL = os.getenv(
    "RPC_URL",
    "https://eth.llamarpc.com"
)

w3 = Web3(Web3.HTTPProvider(RPC_URL))

TRANSACTIONS = {
    "ETH Transfer": 21000,
    "ERC20 Transfer": 65000,
    "NFT Mint": 150000,
    "DEX Swap": 180000,
}

def get_gas_prices():
    latest = w3.eth.get_block("latest")

    base_fee = latest.get("baseFeePerGas", 0)

    samples = []

    for block_number in range(
        max(0, latest.number - 10),
        latest.number
    ):
        try:
            block = w3.eth.get_block(
                block_number,
                full_transactions=True
            )

            for tx in block.transactions:
                if tx.get("gasPrice"):
                    samples.append(tx["gasPrice"])

        except Exception:
            continue

    return base_fee, samples


def analyze_gas():
    if not w3.is_connected():
        raise ConnectionError("RPC connection failed.")

    base_fee, samples = get_gas_prices()

    if not samples:
        print("No gas price samples found.")
        return

    average = statistics.mean(samples)
    median = statistics.median(samples)
    minimum = min(samples)
    maximum = max(samples)

    print("\n========== GAS INTELLIGENCE ==========")

    print(
        f"Network: Ethereum | Chain ID: {w3.eth.chain_id}"
    )

    print(
        f"Block: {w3.eth.block_number}"
    )

    print(
        f"Base Fee: {w3.from_wei(base_fee, 'gwei'):.3f} Gwei"
    )

    print(
        f"Average: {w3.from_wei(average, 'gwei'):.3f} Gwei"
    )

    print(
        f"Median: {w3.from_wei(median, 'gwei'):.3f} Gwei"
    )

    print(
        f"Minimum: {w3.from_wei(minimum, 'gwei'):.3f} Gwei"
    )

    print(
        f"Maximum: {w3.from_wei(maximum, 'gwei'):.3f} Gwei"
    )

    print("\n========== COST ESTIMATES ==========")

    for name, gas_limit in TRANSACTIONS.items():

        cost_wei = gas_limit * average

        cost_eth = w3.from_wei(
            cost_wei,
            "ether"
        )

        print(
            f"{name}: {cost_eth:.8f} ETH"
        )

    print("\n========== NETWORK STATUS ==========")

    if average > w3.to_wei(50, "gwei"):
        print("High gas activity detected.")

    elif average > w3.to_wei(20, "gwei"):
        print("Moderate gas activity detected.")

    else:
        print("Relatively low gas activity.")

    print("=====================================")


if __name__ == "__main__":
    analyze_gas()
