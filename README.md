# TAREFA: Implementar busca semântica (15 min)
# Pode usar Google, documentação, Copilot/Cursor

def semantic_search(query: str, documents: List[str]) -> List[Tuple[str, float]]:
    """
    Retorne os top-5 documentos mais relevantes com scores
    Use qualquer biblioteca/modelo que preferir
    """
    # IMPLEMENTE AQUI
    pass

# DADOS DE TESTE:
docs = [
    "Escritório em Pinheiros com 50m², sala reunião, R$3000",
    "Coworking Vila Olímpia, 24h, estacionamento, R$1500", 
    "Sala privativa Faria Lima, 100m², varanda, R$8000",
    "Estação de trabalho Berrini, ar condicionado, R$800",
    "Escritório Moema, 200m², 3 salas reunião, estacionamento, R$12000"
]

# TESTE:
result = semantic_search("preciso de espaço com estacionamento barato", docs)
print(result)  # Deve retornar Vila Olímpia primeiro
