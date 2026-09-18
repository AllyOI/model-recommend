# Constraints Essay — Concentrate AI Model Recommendation System
 
**Team:** Allison Oh, Aditya Pawar
 
## Economic (Manufacturability)
 
Concentrate AI funds the project's cloud and third-party API costs on an as-needed basis rather than against a fixed dollar cap, which lets us benchmark broadly but still pushes us to meter usage so per-token inference costs on paid APIs like OpenAI's and Anthropic's don't balloon just to add one more candidate model. Most of that spend goes toward running customer-representative prompts against each candidate model rather than toward fixed infrastructure, so the design favors caching and sampled evaluation sets over exhaustively testing every model on every customer workload. This directly trades off against the Security constraint below: the cheapest path is calling third-party APIs directly with customer data, while the safer path of self-hosting open-weight models for evaluation costs more compute time than the discretionary budget comfortably absorbs.
 
## Security
 
Because Concentrate AI's business customers submit real product descriptions and sample prompts, sometimes containing their own customer data, to get a model recommendation, that intake has to be treated as sensitive rather than logged in plaintext or forwarded unredacted to whichever third-party model is being benchmarked. Concretely, this means sanitizing or redacting customer-submitted text before it leaves our pipeline to an external vendor's API, and it is Concentrate AI's business customers specifically who bear the risk if that step is skipped. As noted above, doing this well is the more expensive option relative to a bare API-calling pipeline, so tighter security here pulls directly against the Economic constraint of keeping API spend low.
 
## Social
 
The system's core purpose is helping smaller business customers who lack in-house ML expertise choose a cost-appropriate model instead of overpaying for capability they don't need, which gives it a real public-service angle beyond Concentrate AI's own commercial interest. That benefit only holds up if the recommendation logic explains its reasoning (cost, latency, accuracy trade-offs) rather than acting as a black box, since a customer relying on our recommendation to set a modest monthly AI budget could be genuinely harmed by an opaque bad call.
 
## Legal (Regulatory)
 
Every candidate model we benchmark is reached through its provider's own API terms of service, so we are bound by whatever restrictions vendors like OpenAI and Anthropic place on benchmarking their models and publishing comparative results, and violating those terms in a customer-facing comparison would be a real legal exposure for Concentrate AI. This creates a second trade-off with the Social goal above: the more specifically and openly we show a customer why one vendor's model outperformed another, the closer that comparison gets to the kind of public benchmarking language several providers' usage policies restrict, so our reporting has to stay accurate without becoming a formal head-to-head claim.
