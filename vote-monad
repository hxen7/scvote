#Bloackchain: Monad

from web3 import Web3
from eth_account import Account
import getpass
import sys
import random
import time
import json

def fisher_yates_shuffle(start, end):
    """
    Gera uma lista de números entre start e end e aplica o algoritmo Fisher-Yates
    para embaralhar os intervalos de tempo.
    """
    intervals = list(range(start, end + 1))
    for i in range(len(intervals) - 1, 0, -1):
        j = random.randint(0, i)
        intervals[i], intervals[j] = intervals[j], intervals[i]
    return intervals

def main():
    print("🟢 Iniciando Bot de Votação...")

    # -------------------------------
    # 1) Conectar à rede Monad
    # -------------------------------
    print("🔗 Conectando à rede Monad via RPC...")
    RPC_URL = "https://monad-testnet.g.alchemy.com/v2/0qDSev0jq7JFssu6EA-rZgFzvebFj9xlF"
    web3 = Web3(Web3.HTTPProvider(RPC_URL))
    
    if web3.is_connected():
        print("✅ Conexão bem-sucedida com a rede Monad via RPC!")
    else:
        raise Exception("❌ Falha ao conectar na rede Monad via RPC.")

    # Após confirmar a conexão, obtém o gas price
    try:
        gas_price = web3.eth.gas_price
    except Exception as e:
        print("❌ Erro ao obter gas price. Verifique sua chave de API e conexão.")
        sys.exit(1)
    higher_gas_price = int(gas_price * 1.1)
    
    # -------------------------------
    # 2) Carregar keystore
    # -------------------------------
    keystore_file = input("Digite o caminho do arquivo keystore: ")
    try:
        with open(keystore_file, "r", encoding="utf-8") as keyfile:
            encrypted_key = keyfile.read()
    except FileNotFoundError:
        print(f"❌ Arquivo keystore não encontrado: {keystore_file}")
        sys.exit(1)
    except Exception as e:
        print(f"❌ Erro ao ler arquivo keystore: {e}")
        sys.exit(1)

    password = getpass.getpass("Digite a senha do Keystore: ")

    print("🔐 Descriptografando Keystore...")
    try:
        private_key = Account.decrypt(encrypted_key, password)
        if isinstance(private_key, bytes):
            private_key = private_key.hex()
    except ValueError:
        print("❌ Senha incorreta ou keystore inválido.")
        sys.exit(1)
    except Exception as e:
        print(f"❌ Erro inesperado ao descriptografar keystore: {e}")
        sys.exit(1)

    # -------------------------------
    # 3) Endereço da Conta
    # -------------------------------
    try:
        account = Account.from_key(private_key)
        print(f"🔑 Endereço da conta: {account.address}")
    except Exception as e:
        print(f"❌ Erro ao carregar conta: {e}")
        sys.exit(1)

    # -------------------------------
    # 4) Configuração do contrato
    # -------------------------------
    contract_address = '0x444f67710461d7b7e4da930253066703b5fDeE73'
    contract_address = web3.to_checksum_address(contract_address)
    contract_abi = [
        {"inputs": [], "name": "vote", "outputs": [], "stateMutability": "payable", "type": "function"},
        {"stateMutability": "payable", "type": "receive"}
    ]

    contract = web3.eth.contract(address=contract_address, abi=contract_abi)

    # -------------------------------
    # 5) Função "vote"
    # -------------------------------
    def vote():
        try:
            print("\n🔎 Verificando saldo...")
            balance = web3.eth.get_balance(account.address)
            eth_balance = web3.from_wei(balance, 'ether')
            print(f"💰 Saldo da conta: {eth_balance} ETH")

            if balance == 0:
                raise Exception("❌ Saldo insuficiente para pagar as taxas de rede.")

            print("🔄 Obtendo nonce...")
            nonce = web3.eth.get_transaction_count(account.address)
            print(f"🔄 Nonce atual: {nonce}")

            print("🛠️ Construindo transação...")
            tx = contract.functions.vote().build_transaction({
                'from': account.address,
                'nonce': nonce,
                'value': 0,  # Ajuste se necessário
                'gas': 200000,
                'gasPrice': higher_gas_price,
            })

            print("🔏 Assinando transação...")
            signed_tx = web3.eth.account.sign_transaction(tx, private_key)
            print(f"✍️ Transação assinada com sucesso. Hash: {signed_tx.hash.hex()}")

            print("📤 Enviando transação para a rede...")
            tx_hash = web3.eth.send_raw_transaction(signed_tx.raw_transaction)
            print(f"✅ Transação enviada com sucesso! Hash: {tx_hash.hex()}")

            print("🕒 Aguardando confirmação...")
            receipt = web3.eth.wait_for_transaction_receipt(tx_hash)
            print(f"🎯 Transação confirmada no bloco: {receipt['blockNumber']}")

            return True

        except Exception as e:
            print(f"❌ Erro ao votar: {e}")
            return False

    # -------------------------------
    # 6) Loop principal com múltiplos votos
    # -------------------------------
    try:
        num_votes = int(input("Quantos votos você deseja enviar? "))
        print(f"\n🎲 Iniciando sequência de {num_votes} votos...")

        intervals = fisher_yates_shuffle(30, 60)
        successful_votes = 0

        for i in range(num_votes):
            print(f"\n🗳️ Iniciando voto {i + 1} de {num_votes}")

            if vote():
                successful_votes += 1

                if i < num_votes - 1:
                    wait_time = intervals[i % len(intervals)]
                    print(f"\n⏳ Aguardando {wait_time} segundos antes do próximo voto...")
                    time.sleep(wait_time)

        print(f"\n✅ Processo concluído! Votos bem-sucedidos: {successful_votes}/{num_votes}")

    except ValueError:
        print("❌ Por favor, insira um número válido de votos.")
    except KeyboardInterrupt:
        print("\n\n⚠️ Processo interrompido pelo usuário.")
    except Exception as e:
        print(f"\n❌ Erro inesperado: {e}")

if __name__ == "__main__":
    main()

