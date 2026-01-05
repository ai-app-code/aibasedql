# 🏗️ SUPERBET GENESIS v3.1 - PROJE TREE PART 3
## ML Katmanı (4+ Seviye Derinlik)

---

```
superbet-genesis/ (devam)
│
├── 📁 ml/
│   │
│   ├── 📁 models/
│   │   │
│   │   ├── 📁 layer1_lightgbm/
│   │   │   ├── 📄 __init__.py
│   │   │   ├── 📄 model.py
│   │   │   │   ├── class LightGBMQuantileModel:
│   │   │   │   │   ├── __init__(config: LightGBMConfig):
│   │   │   │   │   │   ├── objective: "quantile"
│   │   │   │   │   │   ├── alpha: 0.7  # Asimetrik risk
│   │   │   │   │   │   ├── boosting_type: "dart"
│   │   │   │   │   │   └── num_boost_round: 200
│   │   │   │   │   ├── train(X, y, eval_set, early_stopping_rounds=50)
│   │   │   │   │   ├── predict(X) -> np.ndarray
│   │   │   │   │   │   └── returns: quantile predictions
│   │   │   │   │   ├── predict_with_variance(X) -> Tuple[np.ndarray, np.ndarray]
│   │   │   │   │   │   ├── predictions: point estimates
│   │   │   │   │   │   └── variance: CAS varyans daraltma için
│   │   │   │   │   ├── get_feature_importance() -> Dict[str, float]
│   │   │   │   │   ├── save(path: str)
│   │   │   │   │   └── load(path: str) -> LightGBMQuantileModel
│   │   │   │   └── FEATURE_COLUMNS: List[str] = [
│   │   │   │       "home_form", "away_form", "h2h_score", "home_xg_avg",
│   │   │   │       "away_xg_avg", "goal_diff", "league_position_diff", ...
│   │   │   │   ]
│   │   │   │
│   │   │   ├── 📄 features.py
│   │   │   │   ├── class FeatureEngineer:
│   │   │   │   │   ├── compute_form_features(match, history) -> FormFeatures
│   │   │   │   │   │   ├── last_5_results: [W, W, D, L, W]
│   │   │   │   │   │   ├── points_last_5: int
│   │   │   │   │   │   ├── goals_scored_last_5: int
│   │   │   │   │   │   └── goals_conceded_last_5: int
│   │   │   │   │   ├── compute_h2h_features(team_a, team_b, n_matches=10)
│   │   │   │   │   │   ├── team_a_wins: int
│   │   │   │   │   │   ├── team_b_wins: int
│   │   │   │   │   │   ├── draws: int
│   │   │   │   │   │   └── avg_goals: float
│   │   │   │   │   ├── compute_league_features(team, league)
│   │   │   │   │   │   ├── position: int
│   │   │   │   │   │   ├── points_per_game: float
│   │   │   │   │   │   └── goal_difference: int
│   │   │   │   │   └── build_feature_vector(match) -> np.ndarray
│   │   │   │   └── FEATURE_CACHE_TTL = 3600  # 1 hour
│   │   │   │
│   │   │   ├── 📄 quantile_config.py
│   │   │   │   ├── @dataclass
│   │   │   │   │   class LightGBMConfig:
│   │   │   │   │   ├── objective: str = "quantile"
│   │   │   │   │   ├── alpha: float = 0.7
│   │   │   │   │   ├── boosting_type: str = "dart"
│   │   │   │   │   ├── learning_rate: float = 0.05
│   │   │   │   │   ├── num_leaves: int = 31
│   │   │   │   │   ├── max_depth: int = 6
│   │   │   │   │   ├── feature_fraction: float = 0.8
│   │   │   │   │   ├── bagging_fraction: float = 0.8
│   │   │   │   │   ├── lambda_l1: float = 0.1
│   │   │   │   │   ├── lambda_l2: float = 0.1
│   │   │   │   │   └── min_data_in_leaf: int = 20
│   │   │   │   └── DYNAMIC_QUANTILE_RANGE = (0.6, 0.9)  # Adaptive α
│   │   │   │
│   │   │   └── 📄 cas_variance.py
│   │   │       └── class CASVarianceReducer:
│   │   │           ├── __init__(target_variance: float = 0.1):
│   │   │           ├── reduce(predictions, raw_variance) -> reduced_variance
│   │   │           │   ├── windsorize: clip extreme values
│   │   │           │   ├── smooth: exponential moving average
│   │   │           │   └── scale: normalize to target range
│   │   │           └── get_confidence_adjustment(variance) -> float
│   │   │
│   │   ├── 📁 layer2_hypernetworks/
│   │   │   ├── 📄 __init__.py
│   │   │   │
│   │   │   ├── 📁 graph_lstm/
│   │   │   │   ├── 📄 encoder.py
│   │   │   │   │   ├── class GraphLSTMEncoder(nn.Module):
│   │   │   │   │   │   ├── __init__(input_dim, hidden_dim, num_gnn_layers, num_lstm_layers):
│   │   │   │   │   │   │   ├── self.gnn = GraphSAGEStack(input_dim, hidden_dim, num_gnn_layers)
│   │   │   │   │   │   │   ├── self.attention_pool = GlobalAttentionPool(hidden_dim)
│   │   │   │   │   │   │   └── self.lstm = nn.LSTM(hidden_dim, hidden_dim, num_lstm_layers, bidirectional=True)
│   │   │   │   │   │   ├── forward(x, edge_index, batch) -> Tuple[Tensor, Tensor]:
│   │   │   │   │   │   │   ├── x_graph = self.gnn(x, edge_index)
│   │   │   │   │   │   │   ├── pooled = self.attention_pool(x_graph, batch)
│   │   │   │   │   │   │   ├── pooled_seq = pooled.view(batch_size, seq_len, -1)
│   │   │   │   │   │   │   ├── lstm_out, (h_n, c_n) = self.lstm(pooled_seq)
│   │   │   │   │   │   │   └── return lstm_out, h_n
│   │   │   │   │   │   └── get_embedding(x, edge_index, batch) -> Tensor [128-dim]
│   │   │   │   │   └── DEFAULT_CONFIG = {"input_dim": 64, "hidden_dim": 128, "num_gnn_layers": 3}
│   │   │   │   │
│   │   │   │   ├── 📄 gnn_layer.py
│   │   │   │   │   ├── class GraphSAGEStack(nn.Module):
│   │   │   │   │   │   ├── __init__(input_dim, hidden_dim, num_layers, aggregator='mean'):
│   │   │   │   │   │   │   └── self.convs = nn.ModuleList([SAGEConv(...) for _ in range(num_layers)])
│   │   │   │   │   │   └── forward(x, edge_index) -> Tensor
│   │   │   │   │   │
│   │   │   │   │   ├── class TemporalGraphNetwork(nn.Module):
│   │   │   │   │   │   ├── __init__(node_dim, edge_dim, memory_dim, time_dim):
│   │   │   │   │   │   │   ├── self.memory = TGNMemory(...)
│   │   │   │   │   │   │   ├── self.message_passing = GraphAttention(...)
│   │   │   │   │   │   │   └── self.time_encoder = TimeEncoder(time_dim)
│   │   │   │   │   │   ├── forward(src, dst, t, msg) -> Tuple[Tensor, Tensor]
│   │   │   │   │   │   └── update_memory(src, dst, t, msg)
│   │   │   │   │   │
│   │   │   │   │   └── class HeterogeneousGraphConv(nn.Module):  # Farklı node/edge tipleri
│   │   │   │   │       ├── node_types: ["team", "player", "match"]
│   │   │   │   │       └── edge_types: ["plays_for", "plays_against", "in_league"]
│   │   │   │   │
│   │   │   │   └── 📄 attention_pool.py
│   │   │   │       └── class GlobalAttentionPool(nn.Module):
│   │   │   │           ├── __init__(hidden_dim):
│   │   │   │           │   └── self.gate_nn = nn.Linear(hidden_dim, 1)
│   │   │   │           └── forward(x, batch) -> Tensor
│   │   │   │               ├── gate = torch.sigmoid(self.gate_nn(x))
│   │   │   │               └── return scatter_add(gate * x, batch, dim=0)
│   │   │   │
│   │   │   ├── 📁 state_space/
│   │   │   │   ├── 📄 core.py
│   │   │   │   │   ├── class LSTMStateSpaceCore(nn.Module):
│   │   │   │   │   │   ├── __init__(input_dim, hidden_dim, state_dim):
│   │   │   │   │   │   │   ├── self.lstm = nn.LSTM(input_dim, hidden_dim, bidirectional=True)
│   │   │   │   │   │   │   ├── self.state_space = StateSpaceModel(input_dim, state_dim)
│   │   │   │   │   │   │   └── self.cross_attn = nn.MultiheadAttention(hidden_dim*2, num_heads=8)
│   │   │   │   │   │   ├── forward(x, h0=None) -> Tuple[Tensor, Tensor]:
│   │   │   │   │   │   │   ├── lstm_out, (h_n, c_n) = self.lstm(x, h0)
│   │   │   │   │   │   │   ├── ssm_out = self.state_space(x)
│   │   │   │   │   │   │   ├── fused, _ = self.cross_attn(lstm_out, ssm_out, ssm_out)
│   │   │   │   │   │   │   └── return fused, h_n
│   │   │   │   │   │   └── get_temporal_dynamics(x) -> Tensor
│   │   │   │   │   └── DEFAULT_CONFIG = {"hidden_dim": 256, "state_dim": 64}
│   │   │   │   │
│   │   │   │   ├── 📄 ssm.py
│   │   │   │   │   └── class StateSpaceModel(nn.Module):
│   │   │   │   │       ├── __init__(input_dim, state_dim):
│   │   │   │   │       │   ├── self.A = nn.Parameter(torch.randn(state_dim, state_dim))
│   │   │   │   │       │   ├── self.B = nn.Linear(input_dim, state_dim)
│   │   │   │   │       │   ├── self.C = nn.Linear(state_dim, input_dim)
│   │   │   │   │       │   └── self.D = nn.Linear(input_dim, input_dim)
│   │   │   │   │       └── forward(x) -> Tensor:
│   │   │   │   │           ├── # x_t+1 = A @ x_t + B @ u_t
│   │   │   │   │           ├── # y_t = C @ x_t + D @ u_t
│   │   │   │   │           └── [discretized implementation]
│   │   │   │   │
│   │   │   │   └── 📄 cross_attention.py
│   │   │   │       └── class CrossModalAttention(nn.Module):
│   │   │   │           ├── __init__(d_model, n_heads, dropout=0.1):
│   │   │   │           └── forward(query, key_value) -> Tensor
│   │   │   │               ├── # Query: temporal features
│   │   │   │               ├── # Key/Value: graph features
│   │   │   │               └── return cross_attended_features
│   │   │   │
│   │   │   └── 📁 tft/
│   │   │       ├── 📄 decoder.py
│   │   │       │   ├── class TemporalFusionDecoder(nn.Module):
│   │   │       │   │   ├── __init__(d_model, n_heads, num_quantiles=3):
│   │   │       │   │   │   ├── self.vsn = VariableSelectionNetwork(...)
│   │   │       │   │   │   ├── self.grn = GatedResidualNetwork(...)
│   │   │       │   │   │   ├── self.attention = InterpretableMultiHeadAttention(...)
│   │   │       │   │   │   └── self.output = nn.Linear(d_model, num_quantiles)
│   │   │       │   │   ├── forward(encoder_output, known_futures, static_context)
│   │   │       │   │   │   ├── selected = self.vsn(encoder_output, static_context)
│   │   │       │   │   │   ├── gated = self.grn(selected)
│   │   │       │   │   │   ├── attended, attention_weights = self.attention(gated)
│   │   │       │   │   │   └── return self.output(attended), attention_weights
│   │   │       │   │   └── get_attention_weights() -> Tensor  # Interpretability
│   │   │       │   └── QUANTILES = [0.1, 0.5, 0.9]
│   │   │       │
│   │   │       ├── 📄 variable_selection.py
│   │   │       │   └── class VariableSelectionNetwork(nn.Module):
│   │   │       │       ├── __init__(d_model, num_inputs, dropout=0.1):
│   │   │       │       │   ├── self.flattened_grn = GRN(num_inputs * d_model, num_inputs)
│   │   │       │       │   └── self.single_grns = nn.ModuleList([GRN(d_model) for _ in range(num_inputs)])
│   │   │       │       └── forward(inputs, context) -> Tuple[Tensor, Tensor]:
│   │   │       │           ├── # Compute variable selection weights
│   │   │       │           ├── weights = softmax(self.flattened_grn(flatten(inputs), context))
│   │   │       │           └── return weighted_sum(inputs, weights), weights
│   │   │       │
│   │   │       └── 📄 interpretable_attention.py
│   │   │           └── class InterpretableMultiHeadAttention(nn.Module):
│   │   │               ├── __init__(d_model, n_heads):
│   │   │               │   └── # Single head but multiple value projections
│   │   │               └── forward(q, k, v) -> Tuple[Tensor, Tensor]:
│   │   │                   ├── # Returns output and interpretable attention weights
│   │   │                   └── return output, attention_weights
│   │   │
│   │   ├── 📁 layer3_edl/
│   │   │   ├── 📄 __init__.py
│   │   │   │
│   │   │   ├── 📄 evidential_head.py
│   │   │   │   ├── class EvidentialHead(nn.Module):
│   │   │   │   │   ├── __init__(input_dim, num_classes=3):
│   │   │   │   │   │   ├── self.fc = nn.Linear(input_dim, num_classes * 4)  # 4 Dirichlet params per class
│   │   │   │   │   │   └── self.softplus = nn.Softplus()
│   │   │   │   │   ├── forward(x) -> Dict[str, Tensor]:
│   │   │   │   │   │   ├── evidence = self.softplus(self.fc(x))  # N × num_classes × 4
│   │   │   │   │   │   ├── alpha = evidence + 1  # Dirichlet concentration
│   │   │   │   │   │   ├── probs = alpha / alpha.sum(dim=-1, keepdim=True)
│   │   │   │   │   │   ├── uncertainty = self._compute_uncertainty(alpha)
│   │   │   │   │   │   ├── if uncertainty.mean() > 0.4:
│   │   │   │   │   │   │   └── return {'action': 'skip', 'uncertainty': uncertainty}
│   │   │   │   │   │   └── return {'probs': probs, 'uncertainty': uncertainty, 'alpha': alpha}
│   │   │   │   │   │
│   │   │   │   │   ├── _compute_uncertainty(alpha) -> Tensor:
│   │   │   │   │   │   ├── S = alpha.sum(dim=-1)  # Dirichlet strength
│   │   │   │   │   │   ├── K = alpha.shape[-1]   # Number of classes
│   │   │   │   │   │   └── return K / S  # Mutual information proxy
│   │   │   │   │   │
│   │   │   │   │   └── get_epistemic_uncertainty(alpha) -> Tensor:
│   │   │   │   │       ├── # Separate aleatoric vs epistemic
│   │   │   │   │       └── return entropy(Dirichlet(alpha)) - expected_entropy
│   │   │   │   │
│   │   │   │   └── UNCERTAINTY_THRESHOLD = 0.4  # τ > 0.4 → skip
│   │   │   │
│   │   │   ├── 📄 loss.py
│   │   │   │   ├── def evidential_loss(pred, target, epoch, max_epochs=100, lambda_kl=0.1):
│   │   │   │   │   ├── alpha = pred['alpha']
│   │   │   │   │   ├── target_one_hot = F.one_hot(target, num_classes=alpha.shape[-1])
│   │   │   │   │   ├── # Negative Log-Likelihood
│   │   │   │   │   ├── L_nll = -torch.sum(target_one_hot * (torch.digamma(alpha) - torch.digamma(alpha.sum(-1, keepdim=True))))
│   │   │   │   │   ├── # KL Annealing
│   │   │   │   │   ├── annealing = min(1.0, epoch / (max_epochs * 0.1))
│   │   │   │   │   ├── uniform_alpha = torch.ones_like(alpha)
│   │   │   │   │   ├── L_kl = kl_divergence_dirichlet(alpha, uniform_alpha)
│   │   │   │   │   └── return L_nll + annealing * lambda_kl * L_kl
│   │   │   │   │
│   │   │   │   └── def kl_divergence_dirichlet(alpha, beta):
│   │   │   │       ├── # KL(Dir(α) || Dir(β))
│   │   │   │       └── [standard Dirichlet KL formula]
│   │   │   │
│   │   │   ├── 📄 calibration.py
│   │   │   │   ├── class EDLCalibrator:
│   │   │   │   │   ├── __init__(temperature=1.0, prior_strength=1.0):
│   │   │   │   │   ├── calibrate_temperature(val_probs, val_labels) -> float:
│   │   │   │   │   │   ├── # Optimize temperature for calibration
│   │   │   │   │   │   └── return optimal_temperature (via L-BFGS)
│   │   │   │   │   │
│   │   │   │   │   ├── compute_ece(probs, labels, n_bins=15) -> float:
│   │   │   │   │   │   ├── # Expected Calibration Error
│   │   │   │   │   │   ├── bins = np.linspace(0, 1, n_bins + 1)
│   │   │   │   │   │   └── return weighted_avg(|accuracy - confidence|)
│   │   │   │   │   │
│   │   │   │   │   ├── pit_uniformity_test(probs, labels) -> PitResult:
│   │   │   │   │   │   ├── # Probability Integral Transform
│   │   │   │   │   │   ├── pit_values = compute_pit(probs, labels)
│   │   │   │   │   │   ├── ks_stat, p_value = stats.kstest(pit_values, 'uniform')
│   │   │   │   │   │   └── return PitResult(ks_stat, p_value, pass=p_value > 0.05)
│   │   │   │   │   │
│   │   │   │   │   └── reliability_diagram(probs, labels, n_bins=10) -> ReliabilityDiagram:
│   │   │   │   │       ├── # Compute calibration curve
│   │   │   │   │       ├── slope, intercept = linear_fit(confidence, accuracy)
│   │   │   │   │       └── return ReliabilityDiagram(slope, intercept)  # slope ≈ 1 is ideal
│   │   │   │   │
│   │   │   │   ├── @dataclass
│   │   │   │   │   class CalibrationMetrics:
│   │   │   │   │   ├── ece: float  # < 0.05 is good
│   │   │   │   │   ├── pit_p_value: float  # > 0.05 is good
│   │   │   │   │   ├── reliability_slope: float  # ≈ 1 is ideal
│   │   │   │   │   └── reliability_intercept: float  # ≈ 0 is ideal
│   │   │   │   │
│   │   │   │   └── CALIBRATION_THRESHOLDS = {
│   │   │   │       "ece": 0.05,
│   │   │   │       "pit_p_value": 0.05,
│   │   │   │       "reliability_slope_min": 0.9,
│   │   │   │       "reliability_slope_max": 1.1
│   │   │   │   }
│   │   │   │
│   │   │   ├── 📄 uncertainty.py
│   │   │   │   ├── class UncertaintyGate:
│   │   │   │   │   ├── __init__(threshold: float = 0.4):
│   │   │   │   │   ├── should_skip(uncertainty: Tensor) -> bool:
│   │   │   │   │   │   └── return uncertainty.mean() > self.threshold
│   │   │   │   │   ├── get_action(uncertainty: Tensor, probs: Tensor) -> Action:
│   │   │   │   │   │   ├── if uncertainty > 0.4: return Action.SKIP
│   │   │   │   │   │   ├── if uncertainty > 0.3: return Action.REDUCE_STAKE
│   │   │   │   │   │   └── return Action.BET
│   │   │   │   │   └── compute_kelly_adjustment(uncertainty: Tensor) -> float:
│   │   │   │   │       ├── # Reduce Kelly fraction based on uncertainty
│   │   │   │   │       └── return 1.0 - uncertainty
│   │   │   │   │
│   │   │   │   └── enum Action: [BET, REDUCE_STAKE, SKIP, HEDGE]
│   │   │   │
│   │   │   └── 📄 config.py
│   │   │       └── @dataclass
│   │   │           class EDLConfig:
│   │   │           ├── input_dim: int = 256
│   │   │           ├── num_classes: int = 3  # [home_win, draw, away_win]
│   │   │           ├── uncertainty_threshold: float = 0.4
│   │   │           ├── kl_annealing_epochs: int = 10
│   │   │           ├── prior_strength: float = 1.0
│   │   │           ├── temperature: float = 1.0
│   │   │           └── calibration_frequency: int = 100  # batches
│   │   │
│   │   ├── 📁 ensemble/
│   │   │   ├── 📄 __init__.py
│   │   │   ├── 📄 ensemble_model.py
│   │   │   │   ├── class DeepEnsemble(nn.Module):
│   │   │   │   │   ├── __init__(base_model_class, num_models=5, **model_kwargs):
│   │   │   │   │   │   └── self.models = nn.ModuleList([base_model_class(**model_kwargs) for _ in range(num_models)])
│   │   │   │   │   ├── forward(x) -> Dict[str, Tensor]:
│   │   │   │   │   │   ├── predictions = [model(x) for model in self.models]
│   │   │   │   │   │   ├── mean_pred = torch.stack(predictions).mean(dim=0)
│   │   │   │   │   │   ├── std_pred = torch.stack(predictions).std(dim=0)
│   │   │   │   │   │   └── return {'prediction': mean_pred, 'uncertainty': std_pred}
│   │   │   │   │   └── get_disagreement() -> Tensor:
│   │   │   │   │       └── return pairwise_disagreement_score
│   │   │   │   └── NUM_ENSEMBLE_MODELS = 5  # Pre-match only (too slow for live)
│   │   │   │
│   │   │   └── 📄 aggregator.py
│   │   │       └── class EnsembleAggregator:
│   │   │           ├── aggregate_predictions(predictions: List[Tensor], method='mean')
│   │   │           │   ├── methods: ['mean', 'median', 'weighted_mean']
│   │   │           │   └── return aggregated_prediction
│   │   │           └── compute_ensemble_uncertainty(predictions) -> Tensor
│   │   │
│   │   └── 📁 hybrid/
│   │       ├── 📄 __init__.py
│   │       ├── 📄 pipeline.py
│   │       │   ├── class HybridModelPipeline:
│   │       │   │   ├── __init__(layer1, layer2, layer3):
│   │       │   │   │   ├── self.layer1 = layer1  # LightGBM-Quantile
│   │       │   │   │   ├── self.layer2 = layer2  # HyperNetworks
│   │       │   │   │   └── self.layer3 = layer3  # EDL
│   │       │   │   ├── forward(raw_features, graph_data) -> HybridPrediction:
│   │       │   │   │   ├── # Layer 1: Quick feature preprocessing
│   │       │   │   │   ├── l1_output = self.layer1.predict_with_variance(raw_features)
│   │       │   │   │   ├── # Layer 2: Deep temporal-spatial encoding
│   │       │   │   │   ├── l2_output = self.layer2(l1_output.features, graph_data)
│   │       │   │   │   ├── # Layer 3: Uncertainty estimation
│   │       │   │   │   ├── l3_output = self.layer3(l2_output)
│   │       │   │   │   └── return HybridPrediction(
│   │       │   │   │       probs=l3_output['probs'],
│   │       │   │   │       uncertainty=l3_output['uncertainty'],
│   │       │   │   │       confidence=l1_output.confidence,
│   │       │   │   │       attention_weights=l2_output.attention_weights
│   │       │   │   │   )
│   │       │   │   └── get_layer_contributions() -> Dict[str, float]
│   │       │   │
│   │       │   └── @dataclass
│   │       │       class HybridPrediction:
│   │       │       ├── probs: Tensor  # [home, draw, away]
│   │       │       ├── uncertainty: float  # τ
│   │       │       ├── confidence: float
│   │       │       ├── attention_weights: Tensor  # Interpretability
│   │       │       └── should_skip: bool  # τ > 0.4
│   │       │
│   │       ├── 📄 inference.py
│   │       │   ├── class HybridInferenceService:
│   │       │   │   ├── __init__(pipeline, feast_client, redis_client):
│   │       │   │   ├── async predict(match_id: int) -> HybridPrediction:
│   │       │   │   │   ├── # Fetch features from Feast
│   │       │   │   │   ├── features = await self.feast_client.get_online_features(match_id)
│   │       │   │   │   ├── # Fetch graph data from Milvus
│   │       │   │   │   ├── graph_data = await self.get_graph_embedding(match_id)
│   │       │   │   │   ├── # Run pipeline
│   │       │   │   │   ├── prediction = self.pipeline(features, graph_data)
│   │       │   │   │   ├── # Cache result
│   │       │   │   │   ├── await self.redis_client.set(f"pred:{match_id}", prediction, ttl=30)
│   │       │   │   │   └── return prediction
│   │       │   │   └── async batch_predict(match_ids: List[int]) -> List[HybridPrediction]
│   │       │   │
│   │       │   └── INFERENCE_TIMEOUT_MS = 40  # p99 < 40ms target
│   │       │
│   │       └── 📄 config.py
│   │           └── @dataclass
│   │               class HybridPipelineConfig:
│   │               ├── layer1_config: LightGBMConfig
│   │               ├── layer2_config: HyperNetworkConfig
│   │               ├── layer3_config: EDLConfig
│   │               ├── use_ensemble_for_prematch: bool = True
│   │               ├── use_edl_for_live: bool = True
│   │               └── max_latency_ms: int = 40
│   │
│   ├── 📁 agents/
│   │   │
│   │   ├── 📁 manager/
│   │   │   ├── 📄 __init__.py
│   │   │   ├── 📄 ucb_manager.py
│   │   │   │   ├── class UCBManagerAgent:
│   │   │   │   │   ├── __init__(workers: List[WorkerAgent], exploration_coef=0.2):
│   │   │   │   │   │   ├── self.workers = workers
│   │   │   │   │   │   ├── self.c = exploration_coef
│   │   │   │   │   │   └── self.arm_stats = {w.name: ArmStats() for w in workers}
│   │   │   │   │   │
│   │   │   │   │   ├── select_worker(state: ManagerState) -> WorkerAgent:
│   │   │   │   │   │   ├── total_pulls = sum(arm.n for arm in self.arm_stats.values())
│   │   │   │   │   │   ├── ucb_scores = {}
│   │   │   │   │   │   ├── for arm_name, arm in self.arm_stats.items():
│   │   │   │   │   │   │   ├── exploitation = arm.q  # Average reward
│   │   │   │   │   │   │   ├── exploration = self.c * sqrt(log(total_pulls) / (arm.n + 1))
│   │   │   │   │   │   │   └── ucb_scores[arm_name] = exploitation + exploration
│   │   │   │   │   │   └── return self.workers[argmax(ucb_scores)]
│   │   │   │   │   │
│   │   │   │   │   ├── update(worker_name: str, reward: float):
│   │   │   │   │   │   ├── arm = self.arm_stats[worker_name]
│   │   │   │   │   │   ├── arm.n += 1
│   │   │   │   │   │   └── arm.q += (reward - arm.q) / arm.n  # Incremental update
│   │   │   │   │   │
│   │   │   │   │   └── get_worker_performance() -> Dict[str, WorkerPerformance]
│   │   │   │   │
│   │   │   │   └── @dataclass
│   │   │   │       class ArmStats:
│   │   │   │       ├── q: float = 0.0  # Average reward
│   │   │   │       ├── n: int = 0      # Number of pulls
│   │   │   │       └── history: deque = field(default_factory=lambda: deque(maxlen=10))
│   │   │   │
│   │   │   ├── 📄 roi_history.py
│   │   │   │   └── class ROIHistoryTracker:
│   │   │   │       ├── __init__(maxlen=10):
│   │   │   │       │   └── self.history = deque(maxlen=maxlen)  # O(1) operations
│   │   │   │       ├── add(roi: float, timestamp: datetime)
│   │   │   │       ├── get_rolling_average() -> float
│   │   │   │       ├── get_trend() -> str  # "UP", "DOWN", "STABLE"
│   │   │   │       └── should_reduce_risk() -> bool:
│   │   │   │           └── return self.get_rolling_average() < -0.02
│   │   │   │
│   │   │   ├── 📄 dynamic_lambda.py
│   │   │   │   └── class DynamicLambdaController:
│   │   │   │       ├── __init__(base_lambda=1.0, mode_multipliers=None):
│   │   │   │       │   └── self.mode_multipliers = mode_multipliers or {
│   │   │   │       │       "COORDINATION": 1.15,
│   │   │   │       │       "LEADERSHIP": 1.40,
│   │   │   │       │       "NEUTRAL": 1.0
│   │   │   │       │   }
│   │   │   │       ├── compute_lambda(gamma: float, roi_history: deque, avg_corr: float) -> float:
│   │   │   │       │   ├── mode = self._detect_mode(gamma)
│   │   │   │       │   ├── mode_mult = self.mode_multipliers[mode]
│   │   │   │       │   ├── corr_adj = 1 + sqrt(avg_corr)
│   │   │   │       │   └── return self.base_lambda * mode_mult * corr_adj
│   │   │   │       └── _detect_mode(gamma) -> str:
│   │   │   │           ├── if gamma < -0.08: return "COORDINATION"
│   │   │   │           ├── if gamma > 0.52: return "LEADERSHIP"
│   │   │   │           └── return "NEUTRAL"
│   │   │   │
│   │   │   └── 📄 worker_selector.py
│   │   │       └── class WorkerSelector:
│   │   │           ├── select(match_status: MatchStatus) -> WorkerType:
│   │   │           │   ├── if match_status == SCHEDULED: return PREMATCH_WORKER
│   │   │           │   ├── if match_status == LIVE: return LIVE_WORKER
│   │   │           │   └── return NONE
│   │   │           └── handover_needed(old_status, new_status) -> bool
│   │   │
│   │   ├── 📁 live_worker/
│   │   │   ├── 📄 ppo_agent.py
│   │   │   │   ├── class PPOLiveAgent:
│   │   │   │   │   ├── __init__(policy_network, value_network, config):
│   │   │   │   │   │   ├── self.policy = policy_network  # LSTM policy
│   │   │   │   │   │   ├── self.value = value_network
│   │   │   │   │   │   ├── self.clip_epsilon = config.clip_epsilon  # 0.2
│   │   │   │   │   │   └── self.entropy_coef = config.entropy_coef  # 0.01
│   │   │   │   │   │
│   │   │   │   │   ├── select_action(state: LiveState, hidden: Tensor) -> Action:
│   │   │   │   │   │   ├── action_probs, new_hidden = self.policy(state, hidden)
│   │   │   │   │   │   ├── # Apply CVaR constraint
│   │   │   │   │   │   ├── constrained_probs = self.cvar_thompson.constrain(action_probs)
│   │   │   │   │   │   └── return sample(constrained_probs), new_hidden
│   │   │   │   │   │
│   │   │   │   │   ├── update(trajectories: List[Trajectory]):
│   │   │   │   │   │   ├── # PPO clipped objective
│   │   │   │   │   │   ├── for _ in range(self.epochs):
│   │   │   │   │   │   │   ├── ratio = new_probs / old_probs
│   │   │   │   │   │   │   ├── clipped = clip(ratio, 1-ε, 1+ε)
│   │   │   │   │   │   │   └── loss = -min(ratio * advantage, clipped * advantage)
│   │   │   │   │   │   └── update_parameters(loss)
│   │   │   │   │   │
│   │   │   │   │   └── get_hidden_state() -> Tensor  # For handover
│   │   │   │   │
│   │   │   │   └── PPO_CONFIG = {"clip_epsilon": 0.2, "epochs": 4, "batch_size": 64}
│   │   │   │
│   │   │   ├── 📄 lstm_policy.py
│   │   │   │   └── class LSTMPolicyNetwork(nn.Module):
│   │   │   │       ├── __init__(state_dim, action_dim, hidden_dim=256):
│   │   │   │       │   ├── self.lstm = nn.LSTM(state_dim, hidden_dim)
│   │   │   │       │   └── self.head = nn.Linear(hidden_dim, action_dim)
│   │   │   │       └── forward(state, hidden) -> Tuple[Tensor, Tensor]:
│   │   │   │           ├── lstm_out, new_hidden = self.lstm(state, hidden)
│   │   │   │           └── return softmax(self.head(lstm_out)), new_hidden
│   │   │   │
│   │   │   ├── 📄 cvar_thompson.py
│   │   │   │   └── class CVaRThompsonSampler:
│   │   │   │       ├── __init__(cvar_limit=0.05):
│   │   │   │       ├── constrained_sample(priors, bankroll) -> Tuple[int, float]:
│   │   │   │       │   ├── samples = [beta.rvs(α, β) for α, β in priors]
│   │   │   │       │   ├── valid = [i for i, s in enumerate(samples) if percentile(s, 5) >= self.cvar_limit]
│   │   │   │       │   ├── if not valid: return None, 0
│   │   │   │       │   ├── best = argmax(samples[valid])
│   │   │   │       │   ├── stake = min(bankroll * 0.05, bankroll * samples[best] * 0.3)
│   │   │   │       │   └── return best, stake
│   │   │   │       └── update_priors(action, outcome)
│   │   │   │
│   │   │   └── 📄 action_space.py
│   │   │       └── class LiveActionSpace:
│   │   │           ├── actions: [BET_HOME, BET_DRAW, BET_AWAY, SKIP, HEDGE, CLOSE_POSITION]
│   │   │           ├── get_stake_range(action) -> Tuple[float, float]
│   │   │           └── is_valid(action, state) -> bool
│   │   │
│   │   ├── 📁 prematch_worker/
│   │   │   ├── 📄 dqn_agent.py
│   │   │   │   ├── class DQNPreMatchAgent:
│   │   │   │   │   ├── __init__(q_network, target_network, replay_buffer, config):
│   │   │   │   │   ├── select_action(state, epsilon) -> Action:
│   │   │   │   │   │   ├── if random() < epsilon: return random_action()
│   │   │   │   │   │   └── return argmax(self.q_network(state))
│   │   │   │   │   ├── update(batch_size=64):
│   │   │   │   │   │   ├── batch = self.replay_buffer.sample(batch_size)
│   │   │   │   │   │   ├── targets = r + γ * max(target_network(s'))
│   │   │   │   │   │   └── loss = MSE(q_network(s, a), targets)
│   │   │   │   │   └── sync_target_network()
│   │   │   │   │
│   │   │   │   └── DQN_CONFIG = {"gamma": 0.99, "epsilon_decay": 0.995, "target_update": 1000}
│   │   │   │
│   │   │   ├── 📄 replay_buffer.py
│   │   │   │   └── class PrioritizedReplayBuffer:
│   │   │   │       ├── __init__(capacity=100000, alpha=0.6):
│   │   │   │       ├── add(state, action, reward, next_state, done, priority)
│   │   │   │       ├── sample(batch_size) -> Tuple[Batch, Weights, Indices]
│   │   │   │       └── update_priorities(indices, priorities)
│   │   │   │
│   │   │   └── 📄 q_network.py
│   │   │       └── class QNetwork(nn.Module):
│   │   │           ├── __init__(state_dim, action_dim, hidden_dims=[256, 128]):
│   │   │           └── forward(state) -> Tensor  # Q-values for each action
│   │   │
│   │   ├── 📁 handover/
│   │   │   ├── 📄 protocol.py
│   │   │   │   ├── class HandoverProtocol:
│   │   │   │   │   ├── __init__(redis_client, timeout_seconds=5):
│   │   │   │   │   ├── execute_handover(match_id, prematch_agent, live_agent):
│   │   │   │   │   │   ├── # 1. Get final state from prematch
│   │   │   │   │   │   ├── pre_state = prematch_agent.get_final_state()
│   │   │   │   │   │   ├── # 2. Atomic transfer via Redis WATCH-MULTI
│   │   │   │   │   │   ├── self.atomic_transfer(match_id, pre_state)
│   │   │   │   │   │   ├── # 3. Initialize live agent with prematch context
│   │   │   │   │   │   ├── live_agent.load_hidden_state(pre_state.hidden_state)
│   │   │   │   │   │   └── # 4. Start teacher forcing
│   │   │   │   │   │   └── self.start_teacher_forcing(live_agent, pre_state)
│   │   │   │   │   └── verify_handover(match_id) -> bool
│   │   │   │   │
│   │   │   │   └── HANDOVER_TIMEOUT_SECONDS = 5
│   │   │   │
│   │   │   ├── 📄 atomic_transfer.py
│   │   │   │   └── class AtomicStateTransfer:
│   │   │   │       ├── transfer(redis, key, state):
│   │   │   │       │   ├── # Redis WATCH-MULTI-EXEC for atomicity
│   │   │   │       │   ├── pipe = redis.pipeline()
│   │   │   │       │   ├── pipe.watch(key)
│   │   │   │       │   ├── pipe.multi()
│   │   │   │       │   ├── pipe.set(key, serialize(state))
│   │   │   │       │   ├── pipe.expire(key, 300)  # 5 min TTL
│   │   │   │       │   └── return pipe.execute()
│   │   │   │       └── rollback(redis, key, previous_state)
│   │   │   │
│   │   │   ├── 📄 teacher_forcing.py
│   │   │   │   └── class TeacherForcingScheduler:
│   │   │   │       ├── __init__(initial_weight=1.0, decay_minutes=10):
│   │   │   │       │   └── self.weight = initial_weight  # 1.0 → 0.0
│   │   │   │       ├── get_current_weight(minutes_elapsed) -> float:
│   │   │   │       │   └── return max(0, 1.0 - minutes_elapsed / self.decay_minutes)
│   │   │   │       └── apply_forcing(live_pred, prematch_pred, weight) -> Tensor:
│   │   │   │           └── return weight * prematch_pred + (1 - weight) * live_pred
│   │   │   │
│   │   │   └── 📄 state_snapshot.py
│   │   │       └── @dataclass
│   │   │           class AgentStateSnapshot:
│   │   │           ├── hidden_state: Tensor  # LSTM hidden state
│   │   │           ├── predictions: Dict[str, float]
│   │   │           ├── confidence: float
│   │   │           ├── position: Optional[Position]
│   │   │           └── timestamp: datetime
│   │   │
│   │   └── 📁 meta_learning/
│   │       ├── 📄 knowledge_distillation.py
│   │       │   ├── class KnowledgeDistiller:
│   │       │   │   ├── __init__(teacher, student, temperature=2.0):
│   │       │   │   ├── distill(teacher_logits, student_logits, hard_labels, epoch, T):
│   │       │   │   │   ├── w_t = 0.3 + 0.7 * sigmoid(epoch - T/2)  # 0.3→1.0
│   │       │   │   │   ├── L_hard = cross_entropy(student_logits, hard_labels)
│   │       │   │   │   ├── L_soft = kl_div(softmax(student_logits/temp), softmax(teacher_logits/temp))
│   │       │   │   │   └── return w_t * L_hard + (1 - w_t) * L_soft
│   │       │   │   └── get_distillation_schedule(p_bcd) -> int:
│   │       │   │       └── return 30 if p_bcd > 0.9 else 60  # matches
│   │       │   │
│   │       │   └── TEMPERATURE = 2.0
│   │       │
│   │       ├── 📄 bcd_detector.py
│   │       │   └── class BayesianChangePointDetector:
│   │       │       ├── __init__(prior_mean=0, prior_var=1):
│   │       │       ├── update(observation: float) -> float:
│   │       │       │   └── return p_changepoint  # [0, 1]
│   │       │       ├── detect_regime_change(gamma_history, roi_history) -> RegimeChangeAlert:
│   │       │       │   ├── p_bcd = self.probability_of_change()
│   │       │       │   ├── gamma_slope = gradient(gamma_history, window=3)
│   │       │       │   ├── if p_bcd > 0.85 and gamma_slope < -0.1:
│   │       │       │   │   └── return RegimeChangeAlert.EARLY_WARNING
│   │       │       │   ├── if p_bcd > 0.92 and roi_drop > 0.15:
│   │       │       │   │   └── return RegimeChangeAlert.OBSERVATION_MODE
│   │       │       │   ├── if p_bcd > 0.95:
│   │       │       │   │   └── return RegimeChangeAlert.PHASE_RESET
│   │       │       │   └── return RegimeChangeAlert.NORMAL
│   │       │       └── get_confidence_interval() -> Tuple[float, float]
│   │       │
│   │       ├── 📄 momentum_transfer.py
│   │       │   └── class MomentumTransfer:
│   │       │       ├── transfer_weights(old_weights, new_weights, delta_gamma) -> Tensor:
│   │       │       │   ├── decay_rate = 0.15 + 0.05 * abs(delta_gamma)
│   │       │       │   └── return (1 - decay_rate) * old_weights + decay_rate * new_weights
│   │       │       └── preserve_momentum(optimizer, decay=0.9):
│   │       │           └── # Keep optimizer momentum buffers
│   │       │
│   │       └── 📄 graduated_response.py
│   │           └── class GraduatedResponseController:
│   │               ├── apply_response(roi_drop: float, current_lambda: float, hard_cap: float):
│   │               │   ├── if roi_drop >= -0.01:
│   │               │   │   └── return current_lambda * 0.85, hard_cap * 0.90
│   │               │   ├── if roi_drop >= -0.015:
│   │               │   │   └── return current_lambda * 0.70, hard_cap * 0.75
│   │               │   ├── if roi_drop >= -0.02:
│   │               │   │   └── return self.rollback_to_previous_regime()
│   │               │   └── return current_lambda, hard_cap
│   │               └── rollback_to_previous_regime() -> Tuple[float, float]
│   │
│   └── 📁 decision/
│       ├── 📄 vsnr.py
│       │   └── class VSNRCalculator:
│       │       ├── calculate(delta_prob: float, variance: float) -> float:
│       │       │   ├── vsnr = abs(delta_prob) / sqrt(variance + 1e-6)
│       │       │   └── return clip(vsnr, 1.5, 3.5)
│       │       └── RANGE = (1.5, 3.5)
│       │
│       ├── 📄 decay.py
│       │   └── class TimeDecayCalculator:
│       │       ├── calculate(minute: int) -> float:
│       │       │   └── return 1 / (1 + exp(0.7 * (minute - 85)))
│       │       └── BREAKPOINT_MINUTE = 85
│       │
│       ├── 📄 cas.py
│       │   └── class CASCalculator:
│       │       ├── calculate(vsnr, decay, confidence_weight, corridor_liquidity) -> float:
│       │       │   └── return (vsnr * decay * confidence_weight) / corridor_liquidity
│       │       ├── get_action(cas: float) -> CASAction:
│       │       │   ├── if cas > 1.0: return TRIGGER_MICRO_CYCLE
│       │       │   ├── if cas >= 0.8: return PREPARE_POSITION
│       │       │   └── return MAINTAIN_WEIGHTS
│       │       └── THRESHOLDS = {"trigger": 1.0, "prepare": 0.8}
│       │
│       ├── 📄 confidence_weight.py
│       │   └── class ConfidenceWeightCalculator:
│       │       ├── calculate(kappa, momentum_corr, vol_idx, depth_ratio) -> float:
│       │       │   ├── raw = 0.4 + 0.6 * tanh(kappa * momentum_corr * vol_idx * (1 + depth_ratio))
│       │       │   └── return clip(raw, 0.4, 1.0)
│       │       ├── update_kappa(target_cas, realized_cas, current_kappa) -> float:
│       │       │   └── return clip(current_kappa + 0.05 * (target - realized), 0.5, 1.5)
│       │       └── RANGE = (0.4, 1.0)
│       │
│       ├── 📄 gamma.py
│       │   └── class GammaCalculator:
│       │       ├── calculate(market_data) -> float
│       │       ├── get_mode(gamma: float) -> MarketMode:
│       │       │   ├── if gamma < -0.08: return COORDINATION  # hysteresis: > -0.05
│       │       │   ├── if gamma > 0.52: return LEADERSHIP    # hysteresis: < 0.48
│       │       │   └── return NEUTRAL
│       │       └── THRESHOLDS = {"coordination": -0.08, "leadership": 0.52}
│       │
│       └── 📄 portfolio_correlation.py
│           └── class PortfolioCorrelationManager:
│               ├── calculate_n_eff(weights: np.ndarray, correlation_matrix: np.ndarray) -> float:
│               │   └── return 1 / (weights.T @ correlation_matrix @ weights)
│               ├── adjust_learning_rate(base_eta, n_eff, k) -> float:
│               │   └── return base_eta * min(1, n_eff / k)
│               └── compute_lambda_adjustment(mode_mult, avg_corr) -> float
```

---

## DEVAMI → PROJECT_TREE_v3.1_PART4.md (Strategy, Risk, Monitoring)
