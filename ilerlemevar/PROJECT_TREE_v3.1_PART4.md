# 🏗️ SUPERBET GENESIS v3.1 - PROJE TREE PART 4
## Strategy, Risk, Monitoring, Security, Data Quality (4+ Seviye Derinlik)

---

```
superbet-genesis/ (devam)
│
├── 📁 strategy/
│   │
│   ├── 📁 coupon_engine/
│   │   ├── 📄 __init__.py
│   │   │
│   │   ├── 📄 integer_programming.py
│   │   │   ├── class OptimalCouponCombinator:
│   │   │   │   ├── __init__(predictions, max_coupons=5, risk_aversion=0.1):
│   │   │   │   │   ├── self.predictions = predictions
│   │   │   │   │   ├── self.max_coupons = max_coupons
│   │   │   │   │   └── self.lambda_ = risk_aversion
│   │   │   │   │
│   │   │   │   ├── solve_optimal_mix(timeout_ms=100) -> List[Coupon]:
│   │   │   │   │   ├── """
│   │   │   │   │   ├── Decision Variables: x[i,j] ∈ {0,1}
│   │   │   │   │   ├── Objective: max Σ E[Return] - λ × Σ Risk
│   │   │   │   │   ├── Constraints:
│   │   │   │   │   ├──   1. Σ_j x[i,j] <= 1  (each prediction in max 1 coupon)
│   │   │   │   │   ├──   2. Correlation[j] <= threshold
│   │   │   │   │   ├──   3. Σ_j Stake[j] <= risk_budget
│   │   │   │   │   ├── """
│   │   │   │   │   ├── problem = pulp.LpProblem("CouponOptimization", pulp.LpMaximize)
│   │   │   │   │   ├── # Binary decision variables
│   │   │   │   │   ├── x = {(i,j): pulp.LpVariable(f"x_{i}_{j}", cat='Binary')
│   │   │   │   │   │       for i in range(len(predictions)) for j in range(max_coupons)}
│   │   │   │   │   ├── # Objective function
│   │   │   │   │   ├── problem += sum(expected_returns) - self.lambda_ * sum(risks)
│   │   │   │   │   ├── # Constraints
│   │   │   │   │   ├── [add_constraint for each prediction]
│   │   │   │   │   ├── problem.solve(pulp.PULP_CBC_CMD(msg=0, timeLimit=timeout_ms/1000))
│   │   │   │   │   └── return self.extract_coupons(x)
│   │   │   │   │
│   │   │   │   ├── _calculate_expected_return(coupon_vars, j) -> float
│   │   │   │   ├── _calculate_risk(coupon_vars, j) -> float
│   │   │   │   └── extract_coupons(x) -> List[Coupon]
│   │   │   │
│   │   │   └── IP_TIMEOUT_MS = 100
│   │   │
│   │   ├── 📄 greedy_optimizer.py
│   │   │   ├── class GreedyCouponOptimizer:
│   │   │   │   ├── __init__(predictions, max_coupon_size=10):
│   │   │   │   ├── optimize(market_context) -> List[Coupon]:
│   │   │   │   │   ├── # Sort predictions by expected value
│   │   │   │   │   ├── sorted_preds = sorted(predictions, key=lambda p: p.ev, reverse=True)
│   │   │   │   │   ├── coupons = []
│   │   │   │   │   ├── for pred in sorted_preds:
│   │   │   │   │   │   ├── best_coupon = self._find_best_coupon(pred, coupons)
│   │   │   │   │   │   └── assign_to_coupon(pred, best_coupon)
│   │   │   │   │   └── return coupons
│   │   │   │   └── _check_correlation_constraint(coupon, new_pred) -> bool
│   │   │   │
│   │   │   └── GREEDY_ACCURACY_LOSS = 0.05  # ~5% vs IP
│   │   │
│   │   ├── 📄 hybrid_optimizer.py
│   │   │   └── class HybridCouponOptimizer:
│   │   │       ├── __init__(ip_optimizer, greedy_optimizer, threshold=10):
│   │   │       ├── optimize(predictions, market_context) -> List[Coupon]:
│   │   │       │   ├── n = len(predictions)
│   │   │       │   ├── if n <= self.threshold:
│   │   │       │   │   ├── logger.info(f"Using IP for {n} predictions")
│   │   │       │   │   └── return self.ip_optimizer.solve_optimal_mix()
│   │   │       │   ├── else:
│   │   │       │   │   ├── logger.info(f"Using Greedy for {n} predictions (IP timeout risk)")
│   │   │       │   │   └── return self.greedy_optimizer.optimize(market_context)
│   │   │       └── _estimate_ip_time(n) -> float  # O(2^n) complexity warning
│   │   │
│   │   ├── 📄 system_coupons.py
│   │   │   ├── class SystemCouponBuilder:
│   │   │   │   ├── SYSTEM_TYPES = {
│   │   │   │   │   ├── "Trixie": {"selections": 3, "bets": 4, "structure": [3, 1]},
│   │   │   │   │   ├── "Patent": {"selections": 3, "bets": 7, "structure": [3, 3, 1]},
│   │   │   │   │   ├── "Yankee": {"selections": 4, "bets": 11, "structure": [6, 4, 1]},
│   │   │   │   │   ├── "Lucky15": {"selections": 4, "bets": 15, "structure": [4, 6, 4, 1]},
│   │   │   │   │   ├── "Lucky31": {"selections": 5, "bets": 31, "structure": "full"},
│   │   │   │   │   ├── "Heinz": {"selections": 6, "bets": 57, "structure": "full"},
│   │   │   │   │   ├── "SuperHeinz": {"selections": 7, "bets": 120, "structure": "full"},
│   │   │   │   │   └── "Goliath": {"selections": 8, "bets": 247, "structure": "full"}
│   │   │   │   │   }
│   │   │   │   ├── build_system(selections, system_type) -> SystemCoupon
│   │   │   │   │   ├── if len(selections) != system_type.selections:
│   │   │   │   │   │   └── raise InvalidSystemError
│   │   │   │   │   └── return generate_all_combinations(selections, system_type)
│   │   │   │   ├── calculate_potential_returns(system_coupon, stake) -> ReturnMatrix
│   │   │   │   └── find_break_even_winners(system_coupon) -> int
│   │   │   │
│   │   │   └── MIN_WINNERS = {"Trixie": 2, "Patent": 1, "Yankee": 2, ...}
│   │   │
│   │   ├── 📄 multi_coupon_kelly.py
│   │   │   ├── class MultiCouponKellySizer:
│   │   │   │   ├── """Edward O. Thorp's Generalized Kelly: f* = Σ^(-1) × μ"""
│   │   │   │   ├── __init__(kelly_fraction=0.25, max_single=0.20):
│   │   │   │   │   ├── self.fraction = kelly_fraction  # Quarter Kelly for safety
│   │   │   │   │   └── self.max_single = max_single
│   │   │   │   │
│   │   │   │   ├── calculate_optimal_stakes(coupon_portfolio) -> np.ndarray:
│   │   │   │   │   ├── # Expected returns (excess)
│   │   │   │   │   ├── mu = np.array([c.expected_return - 1 for c in portfolio])
│   │   │   │   │   ├── # Covariance matrix
│   │   │   │   │   ├── Sigma = self.estimate_coupon_covariance(portfolio)
│   │   │   │   │   ├── # Regularize for numerical stability
│   │   │   │   │   ├── Sigma_reg = Sigma + 0.01 * np.eye(len(portfolio))
│   │   │   │   │   ├── # Generalized Kelly: f* = Σ^(-1) × μ
│   │   │   │   │   ├── f_star = np.linalg.solve(Sigma_reg, mu)
│   │   │   │   │   ├── # Apply constraints
│   │   │   │   │   ├── f_star = np.maximum(f_star, 0)  # No short selling
│   │   │   │   │   ├── if f_star.sum() > 1.0:
│   │   │   │   │   │   └── f_star = f_star / f_star.sum()
│   │   │   │   │   ├── f_star *= self.fraction  # Quarter Kelly
│   │   │   │   │   └── return np.minimum(f_star, self.max_single)
│   │   │   │   │
│   │   │   │   ├── estimate_coupon_covariance(portfolio) -> np.ndarray:
│   │   │   │   │   ├── # Estimate correlation based on shared selections
│   │   │   │   │   └── return correlation_matrix
│   │   │   │   │
│   │   │   │   └── adjust_for_uncertainty(stakes, uncertainty) -> np.ndarray:
│   │   │   │       └── return stakes * (1 - uncertainty)
│   │   │   │
│   │   │   └── KELLY_FRACTION = 0.25  # Conservative
│   │   │
│   │   └── 📄 correlation_matrix.py
│   │       └── class CouponCorrelationEstimator:
│   │           ├── estimate(coupon_a, coupon_b) -> float
│   │           │   ├── # Based on shared league, time proximity, team overlap
│   │           │   └── return correlation
│   │           └── build_matrix(coupons) -> np.ndarray
│   │
│   ├── 📁 strategy_plant/
│   │   ├── 📄 __init__.py
│   │   │
│   │   ├── 📄 contract.py
│   │   │   └── class StrategyPlantContract(ABC):
│   │   │       ├── @abstractmethod
│   │   │       │   def generate_signals(predictions, market_context) -> List[Signal]
│   │   │       ├── @abstractmethod
│   │   │       │   def get_expected_return(signal) -> float
│   │   │       └── @abstractmethod
│   │   │           def get_risk_score(signal) -> float
│   │   │
│   │   ├── 📁 value_based/
│   │   │   ├── 📄 pure_value.py
│   │   │   │   └── class PureValueBetting(StrategyPlantContract):
│   │   │   │       ├── generate_signals(predictions, market_context):
│   │   │   │       │   ├── signals = []
│   │   │   │       │   ├── for pred in predictions:
│   │   │   │       │   │   ├── edge = pred.prob - 1 / pred.odds
│   │   │   │       │   │   ├── if edge > 0:
│   │   │   │       │   │   │   └── signals.append(Signal(pred, edge))
│   │   │   │       │   └── return signals
│   │   │   │       └── NAME = "pure_value"
│   │   │   │
│   │   │   ├── 📄 threshold_value.py
│   │   │   │   └── class ThresholdValueBetting:
│   │   │   │       ├── __init__(edge_min=0.05):
│   │   │   │       └── generate_signals(...):  # Only if edge > 5%
│   │   │   │
│   │   │   └── 📄 adaptive_value.py
│   │   │       └── class AdaptiveValueBetting:
│   │   │           ├── # Adjusts edge threshold based on recent performance
│   │   │           └── _update_threshold(performance)
│   │   │
│   │   ├── 📁 portfolio/
│   │   │   ├── 📄 mean_variance.py
│   │   │   │   └── class MeanVarianceOptimization:
│   │   │   │       ├── optimize(expected_returns, cov_matrix, risk_aversion)
│   │   │   │       └── efficient_frontier(returns, cov, n_points=50)
│   │   │   │
│   │   │   ├── 📄 risk_parity.py
│   │   │   │   └── class RiskParityStrategy:
│   │   │   │       └── # Equal risk contribution from each position
│   │   │   │
│   │   │   └── 📄 stable_markowitz.py
│   │   │       └── class StableMarkowitzOptimizer:
│   │   │           ├── solve(expected_returns, cov_matrix):
│   │   │           │   ├── # Check condition number
│   │   │           │   ├── if np.linalg.cond(cov_matrix) > 1e6:
│   │   │           │   │   └── cov_matrix += 0.01 * np.eye(n)  # Ridge
│   │   │           │   ├── # Eigenvalue thresholding
│   │   │           │   ├── eigenvalues = np.maximum(np.linalg.eigvalsh(cov_matrix), 1e-8)
│   │   │           │   ├── # Cholesky decomposition
│   │   │           │   ├── L = np.linalg.cholesky(cov_matrix)
│   │   │           │   └── return scipy.linalg.solve_triangular(...)
│   │   │           └── RIDGE_LAMBDA = 0.01
│   │   │
│   │   ├── 📁 dynamic/
│   │   │   ├── 📄 momentum.py
│   │   │   ├── 📄 mean_reversion.py
│   │   │   └── 📄 regime_switching.py
│   │   │       └── class RegimeSwitchingStrategy:
│   │   │           ├── detect_regime(market_data) -> Regime
│   │   │           └── switch_sub_strategy(regime) -> Strategy
│   │   │
│   │   └── 📁 ml_based/
│   │       ├── 📄 ensemble_ml.py
│   │       ├── 📄 deep_rl.py
│   │       └── 📄 meta_learning_strategy.py
│   │
│   ├── 📁 allocator/
│   │   ├── 📄 adaptive_allocator.py
│   │   │   └── class AdaptiveStrategyAllocator:
│   │   │       ├── __init__(strategies, initial_alpha=10):
│   │   │       │   └── self.alpha = np.ones(len(strategies)) * initial_alpha  # Dirichlet prior
│   │   │       ├── bayesian_update(strategy_id, return_observed):
│   │   │       │   ├── reward = 1 if return_observed > 0 else 0
│   │   │       │   ├── self.alpha[strategy_id] += reward
│   │   │       │   └── return self.alpha / self.alpha.sum()
│   │   │       ├── thompson_sampling_selection() -> int:
│   │   │       │   ├── sampled_probs = np.random.dirichlet(self.alpha)
│   │   │       │   └── return np.argmax(sampled_probs)
│   │   │       └── get_optimal_allocation() -> np.ndarray
│   │   │
│   │   └── 📄 ab_testing.py
│   │       └── class BayesianABTester:
│   │           ├── test(strategy_a_results, strategy_b_results) -> ABTestResult
│   │           │   ├── posterior_a = beta.rvs(1 + successes_a, 1 + failures_a)
│   │           │   ├── posterior_b = beta.rvs(1 + successes_b, 1 + failures_b)
│   │           │   └── return probability_of_b_better = P(posterior_b > posterior_a)
│   │           └── min_sample_size(effect_size, power=0.8) -> int
│   │
│   └── 📁 execution/
│       ├── 📄 bet_executor.py
│       │   └── class VirtualBetExecutor:
│       │       ├── execute(signal, stake) -> Bet
│       │       │   ├── # Virtual execution (paper trading)
│       │       │   └── return Bet(id, match_id, selection, stake, odds, timestamp)
│       │       └── get_execution_report() -> ExecutionReport
│       │
│       ├── 📄 position_tracker.py
│       │   └── class PositionTracker:
│       │       ├── open_position(bet: Bet)
│       │       ├── close_position(bet_id, result)
│       │       ├── get_open_positions() -> List[Position]
│       │       └── get_exposure() -> Exposure
│       │
│       └── 📄 pnl_calculator.py
│           └── class PnLCalculator:
│               ├── calculate_realized_pnl(closed_positions) -> float
│               ├── calculate_unrealized_pnl(open_positions, current_odds) -> float
│               └── get_pnl_report(period) -> PnLReport
│
├── 📁 risk/
│   │
│   ├── 📁 metrics/
│   │   ├── 📄 var.py
│   │   │   └── class ValueAtRisk:
│   │   │       ├── calculate(returns, confidence=0.95) -> float:
│   │   │       │   └── return np.percentile(returns, (1 - confidence) * 100)
│   │   │       ├── historical_var(returns, window=252)
│   │   │       └── parametric_var(returns, confidence=0.95)
│   │   │
│   │   ├── 📄 cvar.py
│   │   │   └── class ConditionalVaR:
│   │   │       ├── calculate(returns, var_threshold) -> float:
│   │   │       │   └── return np.mean(returns[returns <= var_threshold])
│   │   │       └── expected_shortfall(returns, confidence=0.95)
│   │   │
│   │   ├── 📄 max_drawdown.py
│   │   │   └── class MaxDrawdown:
│   │   │       ├── calculate(equity_curve) -> float:
│   │   │       │   ├── peak = np.maximum.accumulate(equity_curve)
│   │   │       │   ├── drawdown = (equity_curve - peak) / peak
│   │   │       │   └── return np.min(drawdown)
│   │   │       └── get_drawdown_duration(equity_curve) -> int  # days
│   │   │
│   │   ├── 📄 sharpe.py
│   │   │   └── class SharpeRatio:
│   │   │       ├── calculate(returns, risk_free=0.02, periods=252) -> float:
│   │   │       │   ├── excess = returns - risk_free / periods
│   │   │       │   └── return np.sqrt(periods) * np.mean(excess) / np.std(excess)
│   │   │       └── annualize(sharpe, periods=252) -> float
│   │   │
│   │   ├── 📄 sortino.py
│   │   │   └── class SortinoRatio:
│   │   │       └── calculate(returns, target=0) -> float
│   │   │
│   │   └── 📄 kelly.py
│   │       └── class KellyCriterion:
│   │           ├── calculate(win_prob, win_odds) -> float:
│   │           │   ├── b = win_odds - 1
│   │           │   ├── p = win_prob
│   │           │   ├── q = 1 - p
│   │           │   └── return (b * p - q) / b
│   │           ├── fractional_kelly(kelly, fraction=0.75) -> float
│   │           └── FRACTION = 0.75  # Conservative
│   │
│   ├── 📁 limits/
│   │   ├── 📄 config.py
│   │   │   └── RISK_LIMITS = {
│   │   │       ├── "max_single_bet": 0.05,      # 5% of bankroll
│   │   │       ├── "max_daily_loss": 0.10,      # 10% daily loss limit
│   │   │       ├── "max_weekly_loss": 0.20,     # 20% weekly loss limit
│   │   │       ├── "min_odds": 1.20,
│   │   │       ├── "max_odds": 10.0,
│   │   │       └── "max_open_bets": 10
│   │   │   }
│   │   │
│   │   ├── 📄 enforcer.py
│   │   │   └── class RiskLimitEnforcer:
│   │   │       ├── check_single_bet(stake, bankroll) -> bool
│   │   │       ├── check_daily_loss(current_loss, bankroll) -> bool
│   │   │       ├── check_weekly_loss(current_loss, bankroll) -> bool
│   │   │       ├── check_odds_range(odds) -> bool
│   │   │       ├── check_open_positions(current_count) -> bool
│   │   │       └── enforce(action) -> EnforcementResult
│   │   │
│   │   └── 📄 margin_call.py
│   │       └── class MarginCallSystem:
│   │           ├── check_margin(current_exposure, limit) -> MarginStatus
│   │           ├── trigger_margin_call(exposure)
│   │           └── force_close_positions(priority_order)
│   │
│   ├── 📁 circuit_breaker/
│   │   ├── 📄 base.py
│   │   │   └── class CircuitBreaker:
│   │   │       ├── __init__(failure_threshold, timeout_seconds, fallback_fn):
│   │   │       ├── states: [CLOSED, OPEN, HALF_OPEN]
│   │   │       ├── call(fn, *args, **kwargs):
│   │   │       │   ├── if self.state == OPEN:
│   │   │       │   │   └── return self.fallback_fn(*args, **kwargs)
│   │   │       │   ├── try:
│   │   │       │   │   ├── result = fn(*args, **kwargs)
│   │   │       │   │   └── self._on_success()
│   │   │       │   ├── except:
│   │   │       │   │   └── self._on_failure()
│   │   │       │   └── return result
│   │   │       ├── _on_failure():
│   │   │       │   ├── self.failure_count += 1
│   │   │       │   ├── if self.failure_count >= self.threshold:
│   │   │       │   │   └── self.state = OPEN
│   │   │       └── _on_success(): self.failure_count = 0
│   │   │
│   │   ├── 📄 data_plant_breaker.py
│   │   │   └── DataPlantBreaker(failure_threshold=3, timeout=30, fallback=stale_data)
│   │   ├── 📄 intelligence_breaker.py
│   │   │   └── IntelligenceBreaker(threshold=2, timeout=10, fallback=student→redis→rule→skip)
│   │   ├── 📄 feature_store_breaker.py
│   │   │   └── FeatureStoreBreaker(threshold=5, timeout=5, fallback=compute_on_fly)
│   │   ├── 📄 kafka_breaker.py
│   │   │   └── KafkaBreaker(threshold=1, backoff=exponential)
│   │   └── 📄 state_store_breaker.py
│   │       └── StateStoreBreaker(threshold=3, timeout=15, fallback=caffeine_lru)
│   │
│   ├── 📁 emergency/
│   │   ├── 📄 hedge_api.py
│   │   │   └── class EmergencyHedgeAPI:
│   │   │       ├── check_emergency_conditions(state) -> List[str]:
│   │   │       │   ├── triggers = []
│   │   │       │   ├── if open_circuit_breakers >= 3:
│   │   │       │   │   └── triggers.append("circuit_breaker_cascade")
│   │   │       │   ├── if daily_pnl < -0.10:
│   │   │       │   │   └── triggers.append("daily_loss_exceeded")
│   │   │       │   └── return triggers
│   │   │       ├── execute_hedge(positions, urgency):
│   │   │       │   ├── if urgency == "critical":
│   │   │       │   │   └── return self.execute_ioc(positions)
│   │   │       │   └── return self.execute_iceberg(positions, chunk_pct=0.2)
│   │   │       └── get_hedge_cost_estimate(positions) -> float
│   │   │
│   │   ├── 📄 ioc_order.py
│   │   │   └── class ImmediateOrCancelOrder:
│   │   │       └── execute(position) -> ExecutionResult  # Immediate close or cancel
│   │   │
│   │   ├── 📄 iceberg_order.py
│   │   │   └── class IcebergOrder:
│   │   │       ├── __init__(chunk_pct=0.2):  # 20% chunks
│   │   │       └── execute(position) -> List[ExecutionResult]
│   │   │
│   │   └── 📄 cascade_detector.py
│   │       └── class CascadeDetector:
│   │           ├── detect(circuit_breakers) -> bool:
│   │           │   └── return sum(1 for cb in circuit_breakers if cb.state == OPEN) >= 3
│   │           └── get_cascade_severity() -> int
│   │
│   └── 📁 recovery/
│       ├── 📄 state_recovery.py
│       │   └── class StateRecoveryManager:
│       │       ├── save_checkpoint(state):
│       │       │   ├── checkpoint = {
│       │       │   │   ├── "version": self.state_version,
│       │       │   │   ├── "event_offset": kafka.current_offset(),
│       │       │   │   ├── "state": state,
│       │       │   │   └── "crc32": zlib.crc32(json.dumps(state).encode())
│       │       │   │   }
│       │       │   └── self.kafka_producer.send("system.checkpoints", checkpoint)
│       │       ├── load_checkpoint(version=None) -> Checkpoint
│       │       └── verify_integrity(checkpoint) -> bool
│       │
│       ├── 📄 event_replay.py
│       │   └── class EventReplayer:
│       │       ├── replay(start_offset, target_state):
│       │       │   ├── for message in replay_consumer:
│       │       │   │   ├── if not is_event_processed(message.id):
│       │       │   │   │   ├── process_event(message, target_state)
│       │       │   │   │   └── mark_event_processed(message.id)
│       │       └── REPLAY_BATCH_SIZE = 1000
│       │
│       └── 📄 rolling_warm_start.py
│           └── class RollingWarmStartManager:
│               ├── spawn_new_agent(old_weights, overlap=0.8):
│               │   ├── new_agent = create_agent()
│               │   ├── new_agent.load_weights(old_weights * overlap)
│               │   └── return new_agent
│               └── OVERLAP_RATIO = 0.8  # Preserve 80% of learning
│
├── 📁 monitoring/
│   │
│   ├── 📁 prometheus/
│   │   ├── 📄 metrics.py
│   │   │   ├── # Business metrics
│   │   │   ├── prediction_confidence = Gauge("prediction_confidence", "Model confidence")
│   │   │   ├── action_distribution = Histogram("action_distribution", "Action buckets")
│   │   │   ├── roi_per_hour = Gauge("roi_per_hour", "Hourly ROI")
│   │   │   ├── # EDL specific
│   │   │   ├── edl_alpha_sum = Gauge("edl_alpha_sum", "Dirichlet strength")
│   │   │   ├── edl_entropy = Gauge("edl_entropy", "Evidence entropy")
│   │   │   ├── edl_tau = Gauge("edl_tau", "Uncertainty threshold")
│   │   │   ├── edl_skip_rate = Counter("edl_skip_rate", "Skip due to uncertainty")
│   │   │   ├── calibration_ece = Gauge("calibration_ece", "Expected Calibration Error")
│   │   │   ├── # Circuit breaker
│   │   │   ├── circuit_state_change = Counter("circuit_state_change", "CB state transitions")
│   │   │   └── fallback_rate = Counter("fallback_rate", "Fallback invocations")
│   │   │
│   │   └── 📁 rules/
│   │       ├── 📄 slo_alerts.yaml
│   │       │   ├── - alert: InferenceLatencyHigh
│   │       │   │   ├── expr: histogram_quantile(0.99, inference_latency) > 0.04
│   │       │   │   ├── for: 5m
│   │       │   │   └── severity: critical
│   │       │   ├── - alert: CalibrationDegraded
│   │       │   │   ├── expr: calibration_ece > 0.05
│   │       │   │   └── severity: warning
│   │       │   └── - alert: HighSkipRate
│   │       │       ├── expr: rate(edl_skip_rate[5m]) > 0.1
│   │       │       └── severity: warning
│   │       │
│   │       └── 📄 risk_alerts.yaml
│   │           ├── - alert: DailyLossExceeded
│   │           │   └── expr: daily_pnl < -0.10
│   │           └── - alert: CircuitBreakerCascade
│   │               └── expr: sum(circuit_breaker_open) >= 3
│   │
│   ├── 📁 grafana/
│   │   └── 📁 dashboards/
│   │       ├── 📄 overview.json
│   │       │   ├── panels:
│   │       │   │   ├── ROI (daily, weekly, monthly)
│   │       │   │   ├── Win Rate
│   │       │   │   ├── Open Positions
│   │       │   │   ├── Circuit Breaker Status
│   │       │   │   └── System Health
│   │       │   └── variables: [environment, time_range]
│   │       │
│   │       ├── 📄 edl_calibration.json
│   │       │   ├── panels:
│   │       │   │   ├── ECE over time
│   │       │   │   ├── Reliability Diagram
│   │       │   │   ├── PIT Histogram
│   │       │   │   ├── Alpha Sum Distribution
│   │       │   │   ├── Skip Rate
│   │       │   │   └── Latency p50/p95/p99
│   │       │   └── thresholds: {ece: 0.05, pit_p: 0.05, latency_p99: 40ms}
│   │       │
│   │       └── 📄 risk_management.json
│   │           ├── panels:
│   │           │   ├── VaR / CVaR
│   │           │   ├── Max Drawdown
│   │           │   ├── Sharpe / Sortino
│   │           │   ├── Exposure by Market
│   │           │   └── Risk Limit Utilization
│   │           └── alerts: integrated
│   │
│   └── 📁 evidently/
│       ├── 📄 drift_detector.py
│       │   └── class DriftDetector:
│       │       ├── detect_data_drift(reference, current) -> DriftReport
│       │       ├── detect_prediction_drift(reference, current) -> DriftReport
│       │       └── DRIFT_THRESHOLD = 0.1
│       │
│       └── 📄 model_monitor.py
│           └── class ModelPerformanceMonitor:
│               ├── track_accuracy(predictions, actuals)
│               ├── track_calibration(probs, outcomes)
│               └── generate_report() -> EvidentlyReport
│
├── 📁 security/
│   ├── 📁 mtls/
│   │   └── 📄 istio_config.yaml
│   │       ├── PeerAuthentication:
│   │       │   └── mtls.mode: STRICT
│   │       └── AuthorizationPolicy:
│   │           └── rules: [allow specific service-to-service]
│   │
│   ├── 📁 vault/
│   │   ├── 📄 vault_config.hcl
│   │   ├── 📄 app_role.py
│   │   │   └── class VaultAppRole:
│   │   │       ├── authenticate() -> Token
│   │   │       └── get_secret(path) -> Dict
│   │   └── 📄 agent_sidecar.yaml
│   │       └── # Vault Agent Injector config
│   │
│   └── 📁 spiffe/
│       └── 📄 workload_identity.py
│           └── class SpiffeIdentity:
│               ├── get_svid() -> SVID
│               └── validate_peer(svid) -> bool
│
└── 📁 data_quality/
    ├── 📁 great_expectations/
    │   ├── 📁 expectations/
    │   │   ├── 📄 match_data_suite.json
    │   │   │   ├── expect_column_to_exist: ["match_id", "kickoff", "home_team"]
    │   │   │   ├── expect_column_values_to_not_be_null: ["match_id"]
    │   │   │   └── expect_column_values_to_be_between: {"minute": [0, 120]}
    │   │   └── 📄 odds_data_suite.json
    │   │       ├── expect_column_values_to_be_greater_than: {"odds": 1.0}
    │   │       └── expect_table_row_count_to_be_between: [1, 1000000]
    │   │
    │   └── 📄 checkpoints/
    │       └── 📄 hourly_validation.yml
    │
    └── 📁 dbt/
        ├── 📄 dbt_project.yml
        ├── 📁 models/
        │   ├── 📁 staging/
        │   │   └── stg_matches.sql
        │   ├── 📁 intermediate/
        │   │   └── int_team_form.sql
        │   └── 📁 marts/
        │       └── fct_predictions.sql
        │
        └── 📁 tests/
            ├── freshness_test.sql
            │   └── # data_age < 30 seconds
            └── coverage_test.sql
                └── # coverage > 95%
```

---

## 📊 NİHAİ İSTATİSTİKLER

| Kategori | Sayı |
|----------|------|
| **Toplam Klasör** | ~200 |
| **Toplam Dosya** | ~500+ |
| **Python Modülleri** | ~300 |
| **Java Dosyaları** | ~20 |
| **Protobuf Schemas** | 4 |
| **Terraform Modülleri** | 6 |
| **Helm Charts** | 4 |
| **Grafana Dashboards** | 6 |
| **Prometheus Rules** | 10+ |
| **GE Expectation Suites** | 4 |
| **dbt Models** | 10+ |

---

## 🔗 ANA DOSYALAR ÖZET

| Dosya | İçerik |
|-------|--------|
| `PROJECT_TREE_v3.1.md` | Infrastructure (GitHub, ArgoCD, Terraform, Helm) |
| `PROJECT_TREE_v3.1_PART2.md` | Services (data-plant, stream-processor, feature-store, api-gateway) |
| `PROJECT_TREE_v3.1_PART3.md` | ML (models, agents, decision) |
| `PROJECT_TREE_v3.1_PART4.md` | Strategy, Risk, Monitoring, Security, Data Quality |

---

**Versiyon:** SUPERBET GENESIS v3.1 Complete Project Structure  
**Tarih:** 04.01.2026  
**Referans:** bettingenesis-v3.1.md
