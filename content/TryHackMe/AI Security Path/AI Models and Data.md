# Where does Training Data come from ? 
- web scraping - most widely used is "Common Crawl" data set
- licensed datasets - data purchased with platforms, like OpenAI + Reddit, or Meta posts etc.
- synthetic data - LLM-generated data
- internal corpora - company knowledge bases, etc.


## Provenance
- Hard to prove data provenance
	- Where did it come from? 
	- When collected? 
	- Has it been modified? 
  ML- BOM - Machine Learning "Bill of Materials" like SBOM (Software BOM), documents inventory of datasets, licenses, PII categories, filtering decisions etc.

## Building the Model 
Epoch - a complete pass of the training algorithm thru the dataset
Overfitting - when model starts to memorize the data instead of general patterns
- can be due to overtraining, too many epochs
Validation set - the held back portion of training data - used to verify if model is being well trained for patterns rather than memorization. "pop quiz" for AI

|              |                                                                                                              |                                                                                                                                          |
| ------------ | ------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Pruning      | Removes parameters that contribute little to predictions, shrinking model size                               | Changes model behaviour post-training; rarely documented in detail                                                                       |
| Quantisation | Reduces numerical precision of weights (e.g., 32-bit to 8-bit floats) to cut memory and compute requirements | Can degrade safety-aligned behaviour; backdoor defences tested on full-precision models may fail to detect threats in quantised versions |
Federated learning - model is trained decentralized, but each local instance will send weight updates to a centralized server. Increases privacy, but decreases integrity of the training process (harder to verify)


## Pre-trained models
- Like Meta's LLaMA (open weights provided) or GPT (OpenAI, API access)

- can fine-tune these pre-trained models to work on a more specific task, but: 
	- Safety alignment degrades
	- Increases attack surface - too focused on a particular task, loses sight of safety
	- Version matters - using old/vulnerable version of a base pre-trained model ?


## Model Cards
How do we know when to trust a model? Training is opaque.
- Use model cards for some transparency, like below

| Section            | What it should tell you                                                          |
| ------------------ | -------------------------------------------------------------------------------- |
| Training data      | What sources were used, how they were filtered, known gaps or biases             |
| Intended use       | What the model was designed for (and explicitly what it wasn't)                  |
| Evaluation results | Performance metrics across different conditions and demographics                 |
| Known limitations  | Conditions under which the model is known to underperform or behave unexpectedly |
| Bias assessment    | Where training data or evaluation may have introduced skew                       |
| Licence            | What you're legally permitted to do with the model                               |

