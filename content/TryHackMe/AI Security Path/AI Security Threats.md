In order from broad > specific: 
- AI > ML > DL > LLM

* MITRE ATTACK for AI is called MITRE ATLAS

- **Prompt injection** - override or bypass instructions like the system prompt
- **Data Poisoning** - manipulation of the training data itself, to influence / bias the model
- **Model Theft** - gain unauthorized access to private model (ex: query a model API and use outputs to train your own model replicating it)
- **Privacy leakage** - model divulging private information (either data or baked into the model into the weights)
- **Model Drift** - model performance degrades over time as world/needs changes


AI makes these 4 things faster (for blue team)
- Analysis
- Prediction
- Summarization
- Investigation (ingesting raw logs etc.)


Securing AI Models
- RBAC and MFA still apply to reduce attack surface
- Privacy protection - training data still needs to be minimized and encrypted. could be baked into the weights of the model if not properly managed
- AI security standards like ISO/IEC 27090 specific to AI
- Model monitoring for unexpected behaviors, drift etc. Explainability tools like SHAP and LIME

