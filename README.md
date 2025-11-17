# FIRST-PROJECT
This is my first Git Repository
<br>
Author - Aditya Nand
# Clean, reproducible mocked multi-agent demo (paste this cell and run in Kaggle)
from typing import Dict, Any, List
import hashlib
import pprint

# ------------ Agents ------------

def router_agent(query: str) -> Dict[str, str]:
    """Simple keyword-based router returning a route string."""
    q = query.lower()
    if "order" in q or "tracking" in q:
        return {"route": "orders"}
    if "password" in q or "reset" in q:
        return {"route": "account"}
    if "return" in q or "refund" in q or "defect" in q:
        return {"route": "returns"}
    if "warranty" in q:
        return {"route": "warranty"}
    if "student" in q or "discount" in q:
        return {"route": "offers"}
    if "contact" in q or "human" in q or "support" in q:
        return {"route": "support"}
    return {"route": "general"}

def retriever_agent(route: str, query: str) -> Dict[str, str]:
    """Return a short knowledge snippet for a given route."""
    kb = {
        "orders": "Orders ship within 1-2 days. Tracking available 24-48h after shipment.",
        "account": "Password reset: Go to Settings → Reset Password. Check email for reset link.",
        "returns": "Returns accepted within 7 days with photos; refunds processed within 5 business days.",
        "warranty": "1 year limited warranty from date of purchase.",
        "offers": "Student discount: 10% with valid student ID via partner portal.",
        "support": "Human support: Call +91-XXXXXXXXXX or open a ticket; response within 24h.",
        "general": "Payments: Cards, UPI, Netbanking. Some regions support COD."
    }
    return {"snippet": kb.get(route, kb["general"])}

def domain_expert_agent(snippet: str, query: str) -> Dict[str, str]:
    """Compose an answer using the snippet and query. Deterministic additions for orders."""
    answer = f"{snippet} (Answer based on route knowledge.)"
    # Deterministic 'tracking' detail for any order-related queries using hash
    if "order" in query.lower() or "tracking" in query.lower():
        h = hashlib.md5(query.encode()).hexdigest()[:6]
        answer += f" Tracking-ID (mock): TRK{h.upper()}."
    return {"answer": answer}

def qa_checker_agent(answer: str, query: str) -> Dict[str, object]:
    """
    Simple QA checker:
    - If answer contains key route words, mark as ok.
    - Deterministic confidence from hash.
    """
    ok_keywords = ["track", "reset", "returns", "warranty", "discount", "support", "payments"]
    ok = any(k in answer.lower() for k in ok_keywords)
    # deterministic pseudo-confidence
    conf_hash = int(hashlib.sha1((answer + query).encode()).hexdigest()[:8], 16)
    confidence = 0.5 + (conf_hash % 50) / 100  # between 0.5 and ~1.0
    final = answer if ok else "Unable to verify the answer automatically; escalate to human support."
    return {"ok": bool(ok), "confidence": round(confidence, 2), "final": final}

# ------------ Orchestration / Demo Runner ------------

def run_demo(queries: List[str]) -> List[Dict[str, object]]:
    results = []
    for q in queries:
        route = router_agent(q)["route"]
        snippet = retriever_agent(route, q)["snippet"]
        domain_answer = domain_expert_agent(snippet, q)["answer"]
        qa = qa_checker_agent(domain_answer, q)
        results.append({
            "query": q,
            "route": route,
            "snippet": snippet,
            "domain_answer": domain_answer,
            "qa_ok": qa["ok"],
            "final_answer": qa["final"],
            "confidence": qa["confidence"]
        })
    return results

def print_demo_results(results: List[Dict[str, object]]):
    pp = pprint.PrettyPrinter(width=120)
    for i, r in enumerate(results, 1):
        print(f"\n--- Demo #{i} ---")
        print(f"Query    : {r['query']}")
        print(f"Route    : {r['route']}")
        print(f"Snippet  : {r['snippet']}")
        print(f"Answer   : {r['domain_answer']}")
        print(f"QA OK?   : {r['qa_ok']}   Confidence: {r['confidence']}")
        print(f"Final    : {r['final_answer']}")
    print("\n--- End of demo ---\n")

# ------------ Sample queries (10) ------------
demo_queries = [
    "My order #123 hasn't arrived. Where is it?",
    "How do I reset my password?",
    "Can I return opened items?",
    "Do you ship internationally?",
    "I got a defective product, want refund.",
    "What is your warranty period?",
    "Change delivery address after order placed.",
    "Are there student discounts?",
    "How to contact human support?",
    "What payment methods are accepted?"
]

# Run and print
results = run_demo(demo_queries)
print_demo_results(results)
