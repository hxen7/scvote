from web3 import Web3
import json
import sys
import random
import time

def fisher_yates_shuffle(start, end):
    """
    Gera uma lista de números entre start e end e aplica o algoritmo Fisher-Yates
    para embaralhar os intervalos de tempo
    """
    # Criar lista de intervalos
    intervals = list(range(start, end + 1))
    
    # Implementação do Fisher-Yates shuffle
    for i in range(len(intervals) - 1, 0, -1):
        j = random.randint(0, i)
        intervals[i], intervals[j] = intervals[j], intervals[i]
    
    return intervals

def main():
    print("🟢 Iniciando Bot de Votação...")

    # -------------------------------
    # 1) Conectar à rede Sepolia
    # -------------------------------
    print("🔗 Conectando à rede Base via RPC...")
    RPC_URL = "https://base.llamarpc.com"
    web3 = Web3(Web3.HTTPProvider(RPC_URL))

    # Verificar conexão
    if web3.is_connected():
        print("✅ Conexão bem-sucedida com a rede Lisk via RPC!")
    else:
        raise Exception("❌ Falha ao conectar na rede Sepolia via DRPC.")

    # -------------------------------
    # 2) Carregar Keystore
    # -------------------------------
    print("📁 Lendo arquivo de Keystore...")
    keystore_file = r"XXXXX" #adicionar o localização da Keystore
    
    try:
        with open(keystore_file, "r", encoding="utf-8") as keyfile:
            encrypted_key = keyfile.read()
    except FileNotFoundError:
        print(f"❌ Arquivo Keystore não encontrado em: {keystore_file}")
        sys.exit(1)
    except Exception as e:
        print(f"❌ Erro ao ler o arquivo Keystore: {e}")
        sys.exit(1)

    password = input("Digite a senha do Keystore (a senha ficará VISÍVEL enquanto digita): ")

    print("🔐 Tentando descriptografar o Keystore...")    
    try:
        private_key = web3.eth.account.decrypt(encrypted_key, password).hex()
    except ValueError:
        print("❌ Senha incorreta ou Keystore inválido.")
        sys.exit(1)
    except Exception as e:
        print(f"❌ Erro inesperado ao descriptografar o Keystore: {e}")
        sys.exit(1)

    print("🔑 Keystore descriptografado com sucesso!")

    # -------------------------------
    # 3) Endereço da Conta
    # -------------------------------
    account = web3.eth.account.from_key(private_key)
    print(f"🔑 Endereço da conta: {account.address}")

    # -------------------------------
    # 4) Configuração do contrato
    # -------------------------------
    contract_address = 'YYYYY' #adicionar o endereço de contrato
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
            # Passo 1: Verificar Saldo
            print("\n🔎 Verificando saldo...")
            balance = web3.eth.get_balance(account.address)
            eth_balance = web3.from_wei(balance, 'ether')
            print(f"💰 Saldo da conta: {eth_balance} ETH")
            
            if balance == 0:
                raise Exception("❌ Saldo insuficiente para pagar as taxas de rede.")
            
            # Passo 2: Obter Nonce
            print("🔄 Obtendo nonce...")
            nonce = web3.eth.get_transaction_count(account.address)
            print(f"🔄 Nonce atual: {nonce}")
            
            # Passo 3: Construir Transação
            print("🛠️ Construindo transação...")
            tx = contract.functions.vote().build_transaction({
                'from': account.address,
                'nonce': nonce,
                'value': 0,
                'gas': 200000,
                'gasPrice': web3.to_wei('0.05', 'gwei')
            })
            
            print(f"📦 Transação construída: {tx}")
            
            # Passo 4: Assinar Transação
            print("🔏 Assinando transação...")
            signed_tx = web3.eth.account.sign_transaction(tx, private_key)
            print(f"✍️ Transação assinada com sucesso. Hash: {signed_tx.hash.hex()}")
            
            # Passo 5: Enviar Transação
            print("📤 Enviando transação para a rede...")
            tx_hash = web3.eth.send_raw_transaction(signed_tx.raw_transaction)
            print(f"✅ Transação enviada com sucesso! Hash: {tx_hash.hex()}")
            
            # Passo 6: Aguardar Confirmação
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
        
        # Gerar intervalos aleatórios usando Fisher-Yates
        intervals = fisher_yates_shuffle(30, 60)
        successful_votes = 0
        
        for i in range(num_votes):
            print(f"\n🗳️ Iniciando voto {i+1} de {num_votes}")
            
            if vote():
                successful_votes += 1
                
                if i < num_votes - 1:  # Não esperar após o último voto
                    wait_time = intervals[i % len(intervals)]  # Usar módulo para caso num_votes > len(intervals)
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
