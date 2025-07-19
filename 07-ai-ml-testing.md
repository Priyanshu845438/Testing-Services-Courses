
# AI/ML Testing 🤖

## What is AI/ML Testing?

AI/ML testing validates artificial intelligence and machine learning systems for accuracy, fairness, robustness, and reliability. It ensures models perform correctly across different scenarios, detect bias, handle edge cases, and maintain performance over time while meeting ethical standards and business requirements.

Modern AI/ML testing addresses unique challenges including data drift, concept drift, model interpretability, adversarial attacks, bias detection, and continuous monitoring that traditional software testing approaches cannot adequately handle.

## Key Concepts

### Types of AI/ML Testing
- **Model Performance Testing**: Accuracy, precision, recall, F1-score validation
- **Data Quality Testing**: Input data validation and preprocessing verification
- **Model Drift Detection**: Performance degradation over time monitoring
- **Bias and Fairness Testing**: Discriminatory behavior detection and mitigation
- **Robustness Testing**: Adversarial attacks and edge case handling
- **A/B Testing**: Model comparison and performance evaluation
- **Explainability Testing**: Model interpretability and decision transparency

### ML-Specific Testing Challenges
- **Data Drift**: Changes in input data distribution over time
- **Concept Drift**: Changes in target variable relationships
- **Model Interpretability**: Understanding and explaining model decisions
- **Scalability Testing**: Performance under production load
- **Security Testing**: Adversarial attacks, prompt injection, data poisoning
- **Ethical AI Testing**: Ensuring responsible AI practices

## Tools & Frameworks

### Comprehensive Model Testing Framework
```python
# Advanced ML model testing suite
import numpy as np
import pandas as pd
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score
from sklearn.model_selection import cross_val_score, StratifiedKFold
import pytest
import logging
from datetime import datetime
import joblib

class MLModelTestSuite:
    
    def __init__(self, model, X_test, y_test, model_name="ML_Model"):
        """
        Initialize ML model testing suite
        
        Args:
            model: Trained ML model
            X_test: Test features
            y_test: Test labels
            model_name: Name for reporting
        """
        self.model = model
        self.X_test = X_test
        self.y_test = y_test
        self.model_name = model_name
        self.predictions = model.predict(X_test)
        self.test_results = {}
        
        # Setup logging
        logging.basicConfig(level=logging.INFO)
        self.logger = logging.getLogger(__name__)
    
    def test_model_accuracy_threshold(self, min_accuracy=0.8):
        """Test model accuracy meets minimum threshold"""
        accuracy = accuracy_score(self.y_test, self.predictions)
        
        self.logger.info(f"Model accuracy: {accuracy:.4f}")
        
        test_result = {
            'test_name': 'accuracy_threshold',
            'accuracy': accuracy,
            'threshold': min_accuracy,
            'passed': accuracy >= min_accuracy,
            'timestamp': datetime.now().isoformat()
        }
        
        self.test_results['accuracy_threshold'] = test_result
        
        assert accuracy >= min_accuracy, f"Accuracy {accuracy:.3f} below threshold {min_accuracy}"
        return test_result
    
    def test_precision_recall_balance(self, min_precision=0.7, min_recall=0.7):
        """Test precision and recall balance"""
        precision = precision_score(self.y_test, self.predictions, average='weighted')
        recall = recall_score(self.y_test, self.predictions, average='weighted')
        f1 = f1_score(self.y_test, self.predictions, average='weighted')
        
        self.logger.info(f"Precision: {precision:.4f}, Recall: {recall:.4f}, F1: {f1:.4f}")
        
        test_result = {
            'test_name': 'precision_recall_balance',
            'precision': precision,
            'recall': recall,
            'f1_score': f1,
            'min_precision': min_precision,
            'min_recall': min_recall,
            'precision_passed': precision >= min_precision,
            'recall_passed': recall >= min_recall,
            'timestamp': datetime.now().isoformat()
        }
        
        self.test_results['precision_recall_balance'] = test_result
        
        assert precision >= min_precision, f"Precision {precision:.3f} below threshold"
        assert recall >= min_recall, f"Recall {recall:.3f} below threshold"
        
        return test_result
    
    def test_class_imbalance_performance(self):
        """Test performance on imbalanced classes"""
        from sklearn.metrics import classification_report
        
        report = classification_report(self.y_test, self.predictions, output_dict=True)
        
        class_performance = {}
        issues = []
        
        for class_label, metrics in report.items():
            if isinstance(metrics, dict) and 'support' in metrics:
                support = metrics['support']
                f1 = metrics['f1-score']
                precision = metrics['precision']
                recall = metrics['recall']
                
                class_performance[class_label] = {
                    'support': support,
                    'f1_score': f1,
                    'precision': precision,
                    'recall': recall
                }
                
                # Check performance on minority classes
                if support < 100:  # Minority class threshold
                    if f1 < 0.5:
                        issues.append(f"Poor F1 performance on minority class {class_label}: {f1:.3f}")
                    if precision < 0.6:
                        issues.append(f"Low precision on minority class {class_label}: {precision:.3f}")
                    if recall < 0.6:
                        issues.append(f"Low recall on minority class {class_label}: {recall:.3f}")
        
        test_result = {
            'test_name': 'class_imbalance_performance',
            'class_performance': class_performance,
            'issues': issues,
            'passed': len(issues) == 0,
            'timestamp': datetime.now().isoformat()
        }
        
        self.test_results['class_imbalance_performance'] = test_result
        
        if issues:
            self.logger.warning(f"Class imbalance issues found: {issues}")
        
        return test_result
    
    def test_cross_validation_stability(self, cv_folds=5):
        """Test model stability across different data splits"""
        from sklearn.base import clone
        
        # Perform cross-validation
        cv_scores = cross_val_score(
            clone(self.model), 
            self.X_test, 
            self.y_test, 
            cv=StratifiedKFold(n_splits=cv_folds, shuffle=True, random_state=42),
            scoring='accuracy'
        )
        
        cv_mean = np.mean(cv_scores)
        cv_std = np.std(cv_scores)
        cv_min = np.min(cv_scores)
        cv_max = np.max(cv_scores)
        
        self.logger.info(f"CV scores: mean={cv_mean:.4f}, std={cv_std:.4f}, range=[{cv_min:.4f}, {cv_max:.4f}]")
        
        # Stability criteria
        stability_threshold = 0.1
        performance_threshold = 0.75
        
        test_result = {
            'test_name': 'cross_validation_stability',
            'cv_scores': cv_scores.tolist(),
            'cv_mean': cv_mean,
            'cv_std': cv_std,
            'cv_min': cv_min,
            'cv_max': cv_max,
            'stability_threshold': stability_threshold,
            'performance_threshold': performance_threshold,
            'is_stable': cv_std < stability_threshold,
            'good_performance': cv_mean > performance_threshold,
            'passed': cv_std < stability_threshold and cv_mean > performance_threshold,
            'timestamp': datetime.now().isoformat()
        }
        
        self.test_results['cross_validation_stability'] = test_result
        
        assert cv_std < stability_threshold, f"Model unstable: CV std={cv_std:.3f}"
        assert cv_mean > performance_threshold, f"Poor CV performance: mean={cv_mean:.3f}"
        
        return test_result
    
    def test_prediction_confidence(self):
        """Test prediction confidence distribution"""
        if not hasattr(self.model, 'predict_proba'):
            self.logger.warning("Model doesn't support probability predictions")
            return {'test_name': 'prediction_confidence', 'skipped': True}
        
        probabilities = self.model.predict_proba(self.X_test)
        max_probs = np.max(probabilities, axis=1)
        
        # Calculate confidence metrics
        overly_confident_threshold = 0.99
        uncertain_threshold = 0.6
        
        overly_confident = np.sum(max_probs > overly_confident_threshold) / len(max_probs)
        uncertain = np.sum(max_probs < uncertain_threshold) / len(max_probs)
        
        avg_confidence = np.mean(max_probs)
        confidence_std = np.std(max_probs)
        
        test_result = {
            'test_name': 'prediction_confidence',
            'average_confidence': avg_confidence,
            'confidence_std': confidence_std,
            'overly_confident_percentage': overly_confident,
            'uncertain_percentage': uncertain,
            'overly_confident_threshold': overly_confident_threshold,
            'uncertain_threshold': uncertain_threshold,
            'confidence_distribution_healthy': overly_confident < 0.3 and uncertain < 0.4,
            'passed': overly_confident < 0.3 and uncertain < 0.4,
            'timestamp': datetime.now().isoformat()
        }
        
        self.test_results['prediction_confidence'] = test_result
        
        assert overly_confident < 0.3, f"Too many overly confident predictions: {overly_confident:.2%}"
        assert uncertain < 0.4, f"Too many uncertain predictions: {uncertain:.2%}"
        
        return test_result
    
    def test_feature_importance_stability(self):
        """Test feature importance consistency"""
        if not hasattr(self.model, 'feature_importances_'):
            self.logger.warning("Model doesn't support feature importance")
            return {'test_name': 'feature_importance_stability', 'skipped': True}
        
        importances = self.model.feature_importances_
        
        # Check for reasonable distribution of importance
        importance_std = np.std(importances)
        max_importance = np.max(importances)
        num_important_features = np.sum(importances > 0.1)  # Features with >10% importance
        
        test_result = {
            'test_name': 'feature_importance_stability',
            'feature_importances': importances.tolist(),
            'importance_std': importance_std,
            'max_importance': max_importance,
            'num_important_features': int(num_important_features),
            'importance_concentrated': max_importance > 0.8,  # Single feature dominates
            'reasonable_distribution': max_importance < 0.8 and num_important_features >= 3,
            'passed': max_importance < 0.8 and num_important_features >= 3,
            'timestamp': datetime.now().isoformat()
        }
        
        self.test_results['feature_importance_stability'] = test_result
        
        if max_importance > 0.8:
            self.logger.warning(f"Single feature dominates with importance: {max_importance:.3f}")
        
        return test_result
    
    def test_data_leakage_detection(self):
        """Test for potential data leakage indicators"""
        # Check for suspiciously high performance
        accuracy = accuracy_score(self.y_test, self.predictions)
        
        # Check prediction patterns
        unique_predictions = len(np.unique(self.predictions))
        prediction_entropy = -np.sum(np.bincount(self.predictions) / len(self.predictions) * 
                                   np.log(np.bincount(self.predictions) / len(self.predictions) + 1e-10))
        
        test_result = {
            'test_name': 'data_leakage_detection',
            'accuracy': accuracy,
            'unique_predictions': unique_predictions,
            'prediction_entropy': prediction_entropy,
            'suspiciously_high_accuracy': accuracy > 0.99,
            'low_prediction_diversity': unique_predictions < 3,
            'potential_leakage_detected': accuracy > 0.99 or unique_predictions < 3,
            'passed': accuracy <= 0.99 and unique_predictions >= 3,
            'timestamp': datetime.now().isoformat()
        }
        
        self.test_results['data_leakage_detection'] = test_result
        
        if accuracy > 0.99:
            self.logger.warning(f"Suspiciously high accuracy: {accuracy:.4f} - check for data leakage")
        
        return test_result
    
    def generate_comprehensive_report(self):
        """Generate comprehensive test report"""
        total_tests = len(self.test_results)
        passed_tests = sum(1 for result in self.test_results.values() 
                          if result.get('passed', False))
        
        report = {
            'model_name': self.model_name,
            'test_summary': {
                'total_tests': total_tests,
                'passed_tests': passed_tests,
                'failed_tests': total_tests - passed_tests,
                'success_rate': passed_tests / total_tests if total_tests > 0 else 0
            },
            'test_results': self.test_results,
            'overall_status': 'PASS' if passed_tests == total_tests else 'FAIL',
            'report_generated': datetime.now().isoformat()
        }
        
        return report

# Usage example
def run_complete_ml_testing():
    """Example of running complete ML model testing"""
    # Load your trained model and test data
    model = joblib.load('trained_model.pkl')  # Your trained model
    X_test = pd.read_csv('test_features.csv')
    y_test = pd.read_csv('test_labels.csv')['target']
    
    # Initialize test suite
    test_suite = MLModelTestSuite(model, X_test, y_test, "Credit_Risk_Model")
    
    # Run all tests
    try:
        test_suite.test_model_accuracy_threshold(min_accuracy=0.85)
        test_suite.test_precision_recall_balance(min_precision=0.8, min_recall=0.75)
        test_suite.test_class_imbalance_performance()
        test_suite.test_cross_validation_stability()
        test_suite.test_prediction_confidence()
        test_suite.test_feature_importance_stability()
        test_suite.test_data_leakage_detection()
        
        print("All ML model tests passed!")
        
    except AssertionError as e:
        print(f"ML model test failed: {e}")
    
    # Generate comprehensive report
    report = test_suite.generate_comprehensive_report()
    
    # Save report
    with open('ml_test_report.json', 'w') as f:
        import json
        json.dump(report, f, indent=2)
    
    return report
```

### Model Drift Detection Framework
```python
# Advanced model drift detection system
import numpy as np
import pandas as pd
from scipy import stats
from scipy.spatial.distance import jensenshannon
import warnings
import matplotlib.pyplot as plt
import seaborn as sns

class AdvancedModelDriftDetector:
    
    def __init__(self, reference_data, current_data, model, feature_names=None):
        """
        Initialize drift detector
        
        Args:
            reference_data: Training/reference dataset
            current_data: Current production dataset
            model: Trained ML model
            feature_names: List of feature names
        """
        self.reference_data = reference_data
        self.current_data = current_data
        self.model = model
        self.feature_names = feature_names or [f"feature_{i}" for i in range(reference_data.shape[1])]
        self.drift_results = {}
    
    def detect_data_drift(self, significance_level=0.05):
        """Comprehensive data drift detection"""
        drift_results = {}
        
        for i, feature_name in enumerate(self.feature_names):
            ref_feature = self.reference_data.iloc[:, i] if isinstance(self.reference_data, pd.DataFrame) else self.reference_data[:, i]
            curr_feature = self.current_data.iloc[:, i] if isinstance(self.current_data, pd.DataFrame) else self.current_data[:, i]
            
            # Determine if feature is numerical or categorical
            if self._is_numerical_feature(ref_feature):
                drift_result = self._detect_numerical_drift(ref_feature, curr_feature, feature_name, significance_level)
            else:
                drift_result = self._detect_categorical_drift(ref_feature, curr_feature, feature_name, significance_level)
            
            drift_results[feature_name] = drift_result
        
        self.drift_results['data_drift'] = drift_results
        return drift_results
    
    def _is_numerical_feature(self, feature_data):
        """Check if feature is numerical"""
        try:
            pd.to_numeric(feature_data)
            return True
        except (ValueError, TypeError):
            return False
    
    def _detect_numerical_drift(self, ref_data, curr_data, feature_name, significance_level):
        """Detect drift in numerical features using multiple methods"""
        
        # Kolmogorov-Smirnov test
        ks_statistic, ks_p_value = stats.ks_2samp(ref_data, curr_data)
        
        # Mann-Whitney U test
        mw_statistic, mw_p_value = stats.mannwhitneyu(ref_data, curr_data, alternative='two-sided')
        
        # Population Stability Index (PSI)
        psi_score = self._calculate_psi(ref_data, curr_data)
        
        # Jensen-Shannon divergence
        js_distance = self._calculate_js_divergence(ref_data, curr_data)
        
        # Statistical summary comparison
        ref_stats = {
            'mean': np.mean(ref_data),
            'std': np.std(ref_data),
            'median': np.median(ref_data),
            'skewness': stats.skew(ref_data),
            'kurtosis': stats.kurtosis(ref_data)
        }
        
        curr_stats = {
            'mean': np.mean(curr_data),
            'std': np.std(curr_data),
            'median': np.median(curr_data),
            'skewness': stats.skew(curr_data),
            'kurtosis': stats.kurtosis(curr_data)
        }
        
        # Determine drift severity
        drift_detected = (ks_p_value < significance_level or 
                         mw_p_value < significance_level or 
                         psi_score > 0.2 or 
                         js_distance > 0.1)
        
        severity = self._determine_drift_severity(ks_p_value, psi_score, js_distance)
        
        return {
            'feature_type': 'numerical',
            'drift_detected': drift_detected,
            'severity': severity,
            'tests': {
                'kolmogorov_smirnov': {
                    'statistic': ks_statistic,
                    'p_value': ks_p_value,
                    'significant': ks_p_value < significance_level
                },
                'mann_whitney': {
                    'statistic': mw_statistic,
                    'p_value': mw_p_value,
                    'significant': mw_p_value < significance_level
                }
            },
            'metrics': {
                'psi_score': psi_score,
                'jensen_shannon_distance': js_distance
            },
            'statistics': {
                'reference': ref_stats,
                'current': curr_stats
            },
            'timestamp': datetime.now().isoformat()
        }
    
    def _detect_categorical_drift(self, ref_data, curr_data, feature_name, significance_level):
        """Detect drift in categorical features"""
        
        # Get value counts
        ref_counts = pd.Series(ref_data).value_counts()
        curr_counts = pd.Series(curr_data).value_counts()
        
        # Align categories
        all_categories = set(ref_counts.index) | set(curr_counts.index)
        ref_aligned = np.array([ref_counts.get(cat, 0) for cat in all_categories])
        curr_aligned = np.array([curr_counts.get(cat, 0) for cat in all_categories])
        
        # Chi-square test
        try:
            # Add small constant to avoid zero frequencies
            ref_aligned_adj = ref_aligned + 1
            curr_aligned_adj = curr_aligned + 1
            
            chi2_statistic, chi2_p_value = stats.chisquare(curr_aligned_adj, ref_aligned_adj)
        except ValueError:
            chi2_statistic, chi2_p_value = float('inf'), 0.0
        
        # Population Stability Index for categorical
        psi_score = self._calculate_categorical_psi(ref_aligned, curr_aligned)
        
        # Calculate percentage changes
        ref_proportions = ref_aligned / np.sum(ref_aligned) if np.sum(ref_aligned) > 0 else np.zeros_like(ref_aligned)
        curr_proportions = curr_aligned / np.sum(curr_aligned) if np.sum(curr_aligned) > 0 else np.zeros_like(curr_aligned)
        
        # Detect new/missing categories
        new_categories = set(curr_counts.index) - set(ref_counts.index)
        missing_categories = set(ref_counts.index) - set(curr_counts.index)
        
        drift_detected = (chi2_p_value < significance_level or 
                         psi_score > 0.2 or 
                         len(new_categories) > 0 or 
                         len(missing_categories) > 0)
        
        severity = self._determine_drift_severity(chi2_p_value, psi_score, None)
        
        return {
            'feature_type': 'categorical',
            'drift_detected': drift_detected,
            'severity': severity,
            'tests': {
                'chi_square': {
                    'statistic': chi2_statistic,
                    'p_value': chi2_p_value,
                    'significant': chi2_p_value < significance_level
                }
            },
            'metrics': {
                'psi_score': psi_score
            },
            'category_analysis': {
                'reference_categories': list(ref_counts.index),
                'current_categories': list(curr_counts.index),
                'new_categories': list(new_categories),
                'missing_categories': list(missing_categories),
                'reference_proportions': dict(zip(all_categories, ref_proportions)),
                'current_proportions': dict(zip(all_categories, curr_proportions))
            },
            'timestamp': datetime.now().isoformat()
        }
    
    def _calculate_psi(self, reference, current, bins=10):
        """Calculate Population Stability Index"""
        try:
            # Create bins based on reference data
            _, bin_edges = np.histogram(reference, bins=bins)
            
            # Calculate counts for both datasets
            ref_counts, _ = np.histogram(reference, bins=bin_edges)
            curr_counts, _ = np.histogram(current, bins=bin_edges)
            
            # Convert to proportions and add small constant to avoid log(0)
            ref_props = (ref_counts + 1e-6) / np.sum(ref_counts + 1e-6)
            curr_props = (curr_counts + 1e-6) / np.sum(curr_counts + 1e-6)
            
            # Calculate PSI
            psi = np.sum((curr_props - ref_props) * np.log(curr_props / ref_props))
            
            return psi
        except Exception:
            return float('inf')
    
    def _calculate_categorical_psi(self, ref_counts, curr_counts):
        """Calculate PSI for categorical features"""
        try:
            ref_props = (ref_counts + 1e-6) / np.sum(ref_counts + 1e-6)
            curr_props = (curr_counts + 1e-6) / np.sum(curr_counts + 1e-6)
            
            psi = np.sum((curr_props - ref_props) * np.log(curr_props / ref_props))
            return psi
        except Exception:
            return float('inf')
    
    def _calculate_js_divergence(self, ref_data, curr_data, bins=50):
        """Calculate Jensen-Shannon divergence"""
        try:
            # Create histograms
            ref_hist, bin_edges = np.histogram(ref_data, bins=bins, density=True)
            curr_hist, _ = np.histogram(curr_data, bins=bin_edges, density=True)
            
            # Normalize to probability distributions
            ref_hist = ref_hist / np.sum(ref_hist)
            curr_hist = curr_hist / np.sum(curr_hist)
            
            # Add small epsilon to avoid log(0)
            ref_hist += 1e-10
            curr_hist += 1e-10
            
            # Calculate JS divergence
            js_distance = jensenshannon(ref_hist, curr_hist)
            
            return js_distance
        except Exception:
            return float('inf')
    
    def _determine_drift_severity(self, p_value, psi_score, js_distance):
        """Determine drift severity level"""
        if p_value is not None and p_value < 0.001:
            return 'CRITICAL'
        elif psi_score > 0.25:
            return 'HIGH'
        elif psi_score > 0.1 or (js_distance and js_distance > 0.1):
            return 'MEDIUM'
        elif p_value and p_value < 0.05:
            return 'LOW'
        else:
            return 'NONE'
    
    def detect_concept_drift(self, y_reference, y_current, significance_level=0.05):
        """Detect concept drift in target variable"""
        
        # Determine if target is categorical or continuous
        if len(np.unique(y_reference)) <= 10:  # Categorical target
            # Chi-square test for categorical targets
            ref_counts = pd.Series(y_reference).value_counts()
            curr_counts = pd.Series(y_current).value_counts()
            
            all_labels = set(ref_counts.index) | set(curr_counts.index)
            ref_aligned = np.array([ref_counts.get(label, 0) for label in all_labels])
            curr_aligned = np.array([curr_counts.get(label, 0) for label in all_labels])
            
            # Add small constant to avoid zero frequencies
            ref_aligned += 1
            curr_aligned += 1
            
            statistic, p_value = stats.chisquare(curr_aligned, ref_aligned)
            test_type = 'chi_square'
            
        else:  # Continuous target
            # Kolmogorov-Smirnov test for continuous targets
            statistic, p_value = stats.ks_2samp(y_reference, y_current)
            test_type = 'kolmogorov_smirnov'
        
        # Calculate target statistics
        ref_stats = {
            'mean': np.mean(y_reference),
            'std': np.std(y_reference),
            'median': np.median(y_reference),
            'min': np.min(y_reference),
            'max': np.max(y_reference)
        }
        
        curr_stats = {
            'mean': np.mean(y_current),
            'std': np.std(y_current),
            'median': np.median(y_current),
            'min': np.min(y_current),
            'max': np.max(y_current)
        }
        
        concept_drift_result = {
            'concept_drift_detected': p_value < significance_level,
            'test_type': test_type,
            'statistic': statistic,
            'p_value': p_value,
            'significance_level': significance_level,
            'target_statistics': {
                'reference': ref_stats,
                'current': curr_stats
            },
            'timestamp': datetime.now().isoformat()
        }
        
        self.drift_results['concept_drift'] = concept_drift_result
        return concept_drift_result
    
    def detect_model_performance_drift(self, y_true_reference, y_true_current, threshold=0.05):
        """Detect drift in model performance"""
        
        # Get predictions for both datasets
        ref_predictions = self.model.predict(self.reference_data)
        curr_predictions = self.model.predict(self.current_data)
        
        # Calculate performance metrics
        ref_accuracy = accuracy_score(y_true_reference, ref_predictions)
        curr_accuracy = accuracy_score(y_true_current, curr_predictions)
        
        ref_f1 = f1_score(y_true_reference, ref_predictions, average='weighted')
        curr_f1 = f1_score(y_true_current, curr_predictions, average='weighted')
        
        # Calculate performance degradation
        accuracy_degradation = ref_accuracy - curr_accuracy
        f1_degradation = ref_f1 - curr_f1
        
        # Check if model supports probability predictions
        performance_drift_details = {
            'reference_accuracy': ref_accuracy,
            'current_accuracy': curr_accuracy,
            'accuracy_degradation': accuracy_degradation,
            'reference_f1': ref_f1,
            'current_f1': curr_f1,
            'f1_degradation': f1_degradation
        }
        
        if hasattr(self.model, 'predict_proba'):
            # Compare prediction confidence distributions
            ref_probs = self.model.predict_proba(self.reference_data)
            curr_probs = self.model.predict_proba(self.current_data)
            
            # Calculate average prediction confidence
            ref_confidence = np.mean(np.max(ref_probs, axis=1))
            curr_confidence = np.mean(np.max(curr_probs, axis=1))
            
            # Jensen-Shannon divergence for probability distributions
            js_distances = []
            for i in range(ref_probs.shape[1]):
                ref_hist, bins = np.histogram(ref_probs[:, i], bins=20, range=(0, 1), density=True)
                curr_hist, _ = np.histogram(curr_probs[:, i], bins=bins, density=True)
                
                ref_hist = ref_hist / np.sum(ref_hist)
                curr_hist = curr_hist / np.sum(curr_hist)
                
                ref_hist += 1e-10
                curr_hist += 1e-10
                
                js_dist = jensenshannon(ref_hist, curr_hist)
                js_distances.append(js_dist)
            
            avg_js_distance = np.mean(js_distances)
            
            performance_drift_details.update({
                'reference_confidence': ref_confidence,
                'current_confidence': curr_confidence,
                'confidence_degradation': ref_confidence - curr_confidence,
                'prediction_distribution_drift': avg_js_distance
            })
            
            # Determine if performance drift occurred
            performance_drift_detected = (
                accuracy_degradation > threshold or
                f1_degradation > threshold or
                avg_js_distance > 0.1
            )
        else:
            performance_drift_detected = (
                accuracy_degradation > threshold or
                f1_degradation > threshold
            )
        
        performance_drift_result = {
            'performance_drift_detected': performance_drift_detected,
            'degradation_threshold': threshold,
            'metrics': performance_drift_details,
            'timestamp': datetime.now().isoformat()
        }
        
        self.drift_results['performance_drift'] = performance_drift_result
        return performance_drift_result
    
    def generate_drift_report(self):
        """Generate comprehensive drift analysis report"""
        
        # Count drift occurrences
        data_drift_count = 0
        if 'data_drift' in self.drift_results:
            data_drift_count = sum(1 for result in self.drift_results['data_drift'].values() 
                                 if result['drift_detected'])
        
        concept_drift_detected = self.drift_results.get('concept_drift', {}).get('concept_drift_detected', False)
        performance_drift_detected = self.drift_results.get('performance_drift', {}).get('performance_drift_detected', False)
        
        # Determine overall drift status
        overall_drift_detected = (data_drift_count > 0 or 
                                concept_drift_detected or 
                                performance_drift_detected)
        
        # Create summary
        drift_summary = {
            'overall_drift_detected': overall_drift_detected,
            'data_drift_features': data_drift_count,
            'total_features_analyzed': len(self.feature_names),
            'concept_drift_detected': concept_drift_detected,
            'performance_drift_detected': performance_drift_detected,
            'drift_severity': self._calculate_overall_severity()
        }
        
        report = {
            'model_drift_analysis': {
                'summary': drift_summary,
                'detailed_results': self.drift_results,
                'recommendations': self._generate_recommendations(),
                'analysis_timestamp': datetime.now().isoformat()
            }
        }
        
        return report
    
    def _calculate_overall_severity(self):
        """Calculate overall drift severity"""
        if 'data_drift' not in self.drift_results:
            return 'UNKNOWN'
        
        severities = [result['severity'] for result in self.drift_results['data_drift'].values()]
        
        if 'CRITICAL' in severities:
            return 'CRITICAL'
        elif 'HIGH' in severities:
            return 'HIGH'
        elif 'MEDIUM' in severities:
            return 'MEDIUM'
        elif 'LOW' in severities:
            return 'LOW'
        else:
            return 'NONE'
    
    def _generate_recommendations(self):
        """Generate actionable recommendations based on drift analysis"""
        recommendations = []
        
        if 'data_drift' in self.drift_results:
            drift_features = [name for name, result in self.drift_results['data_drift'].items() 
                            if result['drift_detected']]
            
            if drift_features:
                recommendations.append(f"Data drift detected in {len(drift_features)} features: {drift_features}")
                recommendations.append("Consider retraining the model with recent data")
                
                high_severity_features = [name for name, result in self.drift_results['data_drift'].items() 
                                        if result['severity'] in ['CRITICAL', 'HIGH']]
                
                if high_severity_features:
                    recommendations.append(f"High severity drift in features: {high_severity_features}")
                    recommendations.append("Immediate model retraining recommended")
        
        if self.drift_results.get('concept_drift', {}).get('concept_drift_detected', False):
            recommendations.append("Concept drift detected - target variable distribution has changed")
            recommendations.append("Review data collection process and retrain model")
        
        if self.drift_results.get('performance_drift', {}).get('performance_drift_detected', False):
            recommendations.append("Model performance has degraded")
            recommendations.append("Implement model retraining pipeline or rollback to previous version")
        
        if not recommendations:
            recommendations.append("No significant drift detected - continue monitoring")
        
        return recommendations

# Usage example
def run_comprehensive_drift_detection():
    """Example of running comprehensive drift detection"""
    # Load reference and current datasets
    reference_data = pd.read_csv('reference_data.csv')
    current_data = pd.read_csv('current_data.csv')
    model = joblib.load('trained_model.pkl')
    
    # Initialize drift detector
    drift_detector = AdvancedModelDriftDetector(
        reference_data.drop('target', axis=1),
        current_data.drop('target', axis=1),
        model,
        feature_names=reference_data.drop('target', axis=1).columns.tolist()
    )
    
    # Detect different types of drift
    data_drift_results = drift_detector.detect_data_drift(significance_level=0.05)
    concept_drift_results = drift_detector.detect_concept_drift(
        reference_data['target'], 
        current_data['target']
    )
    performance_drift_results = drift_detector.detect_model_performance_drift(
        reference_data['target'],
        current_data['target'],
        threshold=0.05
    )
    
    # Generate comprehensive report
    drift_report = drift_detector.generate_drift_report()
    
    # Print summary
    print("=== Model Drift Detection Results ===")
    summary = drift_report['model_drift_analysis']['summary']
    print(f"Overall Drift Detected: {summary['overall_drift_detected']}")
    print(f"Data Drift Features: {summary['data_drift_features']}/{summary['total_features_analyzed']}")
    print(f"Concept Drift: {summary['concept_drift_detected']}")
    print(f"Performance Drift: {summary['performance_drift_detected']}")
    print(f"Severity: {summary['drift_severity']}")
    
    print("\n=== Recommendations ===")
    for rec in drift_report['model_drift_analysis']['recommendations']:
        print(f"- {rec}")
    
    # Save detailed report
    with open('drift_analysis_report.json', 'w') as f:
        import json
        json.dump(drift_report, f, indent=2)
    
    return drift_report
```

### Bias and Fairness Testing Framework
```python
# Comprehensive bias and fairness testing
import numpy as np
import pandas as pd
from sklearn.metrics import confusion_matrix, precision_score, recall_score
import matplotlib.pyplot as plt
import seaborn as sns

class ComprehensiveFairnessTestSuite:
    
    def __init__(self, model, X_test, y_test, sensitive_features, feature_names=None):
        """
        Initialize fairness testing suite
        
        Args:
            model: Trained ML model
            X_test: Test features
            y_test: Test labels
            sensitive_features: Dict mapping sensitive attribute names to column indices/names
            feature_names: List of feature names
        """
        self.model = model
        self.X_test = X_test
        self.y_test = y_test
        self.sensitive_features = sensitive_features
        self.feature_names = feature_names
        self.predictions = model.predict(X_test)
        self.fairness_results = {}
        
        # Get probability predictions if available
        if hasattr(model, 'predict_proba'):
            self.prediction_probabilities = model.predict_proba(X_test)
        else:
            self.prediction_probabilities = None
    
    def test_demographic_parity(self, threshold=0.1):
        """Test for demographic parity across sensitive groups"""
        results = {}
        
        for feature_name, feature_key in self.sensitive_features.items():
            if isinstance(feature_key, str):
                # Feature name provided
                sensitive_values = self.X_test[feature_key] if isinstance(self.X_test, pd.DataFrame) else self.X_test[:, self.feature_names.index(feature_key)]
            else:
                # Column index provided
                sensitive_values = self.X_test.iloc[:, feature_key] if isinstance(self.X_test, pd.DataFrame) else self.X_test[:, feature_key]
            
            groups = np.unique(sensitive_values)
            group_rates = {}
            group_sizes = {}
            
            for group in groups:
                group_mask = sensitive_values == group
                group_predictions = self.predictions[group_mask]
                group_y_true = self.y_test[group_mask]
                
                # Calculate positive prediction rate
                positive_rate = np.mean(group_predictions == 1) if len(group_predictions) > 0 else 0
                group_rates[str(group)] = positive_rate
                group_sizes[str(group)] = len(group_predictions)
            
            # Calculate demographic parity metrics
            max_rate = max(group_rates.values())
            min_rate = min(group_rates.values())
            max_difference = max_rate - min_rate
            
            # Calculate relative difference
            relative_difference = max_difference / max_rate if max_rate > 0 else 0
            
            results[feature_name] = {
                'group_rates': group_rates,
                'group_sizes': group_sizes,
                'max_difference': max_difference,
                'relative_difference': relative_difference,
                'passes_threshold': max_difference <= threshold,
                'fairness_metric': 'demographic_parity',
                'threshold': threshold
            }
        
        self.fairness_results['demographic_parity'] = results
        return results
    
    def test_equalized_odds(self, threshold=0.1):
        """Test for equalized odds (TPR and FPR equality across groups)"""
        results = {}
        
        for feature_name, feature_key in self.sensitive_features.items():
            if isinstance(feature_key, str):
                sensitive_values = self.X_test[feature_key] if isinstance(self.X_test, pd.DataFrame) else self.X_test[:, self.feature_names.index(feature_key)]
            else:
                sensitive_values = self.X_test.iloc[:, feature_key] if isinstance(self.X_test, pd.DataFrame) else self.X_test[:, feature_key]
            
            groups = np.unique(sensitive_values)
            group_metrics = {}
            
            for group in groups:
                group_mask = sensitive_values == group
                group_y_true = self.y_test[group_mask]
                group_y_pred = self.predictions[group_mask]
                
                if len(group_y_true) == 0:
                    continue
                
                # Calculate confusion matrix
                try:
                    tn, fp, fn, tp = confusion_matrix(group_y_true, group_y_pred, labels=[0, 1]).ravel()
                except ValueError:
                    # Handle case where only one class is present
                    if len(np.unique(group_y_true)) == 1:
                        if group_y_true[0] == 1:
                            tp = np.sum(group_y_pred == 1)
                            fn = np.sum(group_y_pred == 0)
                            tn = fp = 0
                        else:
                            tn = np.sum(group_y_pred == 0)
                            fp = np.sum(group_y_pred == 1)
                            tp = fn = 0
                    else:
                        continue
                
                # Calculate rates
                tpr = tp / (tp + fn) if (tp + fn) > 0 else 0  # True Positive Rate (Sensitivity)
                fpr = fp / (fp + tn) if (fp + tn) > 0 else 0  # False Positive Rate
                tnr = tn / (tn + fp) if (tn + fp) > 0 else 0  # True Negative Rate (Specificity)
                fnr = fn / (fn + tp) if (fn + tp) > 0 else 0  # False Negative Rate
                
                group_metrics[str(group)] = {
                    'tpr': tpr,
                    'fpr': fpr,
                    'tnr': tnr,
                    'fnr': fnr,
                    'confusion_matrix': {'tp': int(tp), 'fp': int(fp), 'tn': int(tn), 'fn': int(fn)},
                    'group_size': len(group_y_true)
                }
            
            # Calculate differences between groups
            if len(group_metrics) >= 2:
                tpr_values = [metrics['tpr'] for metrics in group_metrics.values()]
                fpr_values = [metrics['fpr'] for metrics in group_metrics.values()]
                
                tpr_diff = max(tpr_values) - min(tpr_values)
                fpr_diff = max(fpr_values) - min(fpr_values)
                
                # Equalized odds satisfied if both TPR and FPR differences are small
                equalized_odds_satisfied = tpr_diff <= threshold and fpr_diff <= threshold
            else:
                tpr_diff = fpr_diff = 0
                equalized_odds_satisfied = True
            
            results[feature_name] = {
                'group_metrics': group_metrics,
                'tpr_difference': tpr_diff,
                'fpr_difference': fpr_diff,
                'passes_threshold': equalized_odds_satisfied,
                'fairness_metric': 'equalized_odds',
                'threshold': threshold
            }
        
        self.fairness_results['equalized_odds'] = results
        return results
    
    def test_equality_of_opportunity(self, threshold=0.1):
        """Test equality of opportunity (TPR equality across groups)"""
        results = {}
        
        for feature_name, feature_key in self.sensitive_features.items():
            if isinstance(feature_key, str):
                sensitive_values = self.X_test[feature_key] if isinstance(self.X_test, pd.DataFrame) else self.X_test[:, self.feature_names.index(feature_key)]
            else:
                sensitive_values = self.X_test.iloc[:, feature_key] if isinstance(self.X_test, pd.DataFrame) else self.X_test[:, feature_key]
            
            groups = np.unique(sensitive_values)
            group_tpr = {}
            
            for group in groups:
                group_mask = sensitive_values == group
                group_y_true = self.y_test[group_mask]
                group_y_pred = self.predictions[group_mask]
                
                # Focus only on positive examples (y_true == 1)
                positive_mask = group_y_true == 1
                if np.sum(positive_mask) == 0:
                    group_tpr[str(group)] = 0
                    continue
                
                positive_y_true = group_y_true[positive_mask]
                positive_y_pred = group_y_pred[positive_mask]
                
                # Calculate TPR for positive examples
                tpr = np.mean(positive_y_pred == 1)
                group_tpr[str(group)] = tpr
            
            # Calculate TPR difference
            if len(group_tpr) >= 2:
                tpr_values = list(group_tpr.values())
                tpr_diff = max(tpr_values) - min(tpr_values)
                equality_satisfied = tpr_diff <= threshold
            else:
                tpr_diff = 0
                equality_satisfied = True
            
            results[feature_name] = {
                'group_tpr': group_tpr,
                'tpr_difference': tpr_diff,
                'passes_threshold': equality_satisfied,
                'fairness_metric': 'equality_of_opportunity',
                'threshold': threshold
            }
        
        self.fairness_results['equality_of_opportunity'] = results
        return results
    
    def test_individual_fairness(self, distance_threshold=0.1, prediction_threshold=0.1, sample_size=1000):
        """Test individual fairness - similar individuals should get similar predictions"""
        from sklearn.metrics.pairwise import euclidean_distances
        from sklearn.preprocessing import StandardScaler
        
        # Sample data for computational efficiency
        if len(self.X_test) > sample_size:
            indices = np.random.choice(len(self.X_test), sample_size, replace=False)
            X_sample = self.X_test.iloc[indices] if isinstance(self.X_test, pd.DataFrame) else self.X_test[indices]
            y_sample = self.y_test.iloc[indices] if isinstance(self.y_test, pd.Series) else self.y_test[indices]
        else:
            X_sample = self.X_test
            y_sample = self.y_test
        
        # Exclude sensitive features from similarity calculation
        sensitive_indices = []
        for feature_key in self.sensitive_features.values():
            if isinstance(feature_key, str) and self.feature_names:
                sensitive_indices.append(self.feature_names.index(feature_key))
            elif isinstance(feature_key, int):
                sensitive_indices.append(feature_key)
        
        # Get non-sensitive features
        if isinstance(X_sample, pd.DataFrame):
            non_sensitive_features = X_sample.drop(columns=[col for col in X_sample.columns if col in self.sensitive_features.values()])
        else:
            non_sensitive_mask = np.ones(X_sample.shape[1], dtype=bool)
            non_sensitive_mask[sensitive_indices] = False
            non_sensitive_features = X_sample[:, non_sensitive_mask]
        
        # Select only numeric features for distance calculation
        if isinstance(non_sensitive_features, pd.DataFrame):
            numeric_features = non_sensitive_features.select_dtypes(include=[np.number])
        else:
            numeric_features = non_sensitive_features
        
        if numeric_features.shape[1] == 0:
            return {
                'error': 'No numeric features available for individual fairness testing',
                'fairness_metric': 'individual_fairness'
            }
        
        # Standardize features
        scaler = StandardScaler()
        scaled_features = scaler.fit_transform(numeric_features)
        
        # Calculate pairwise distances
        distances = euclidean_distances(scaled_features)
        
        # Get predictions for sample
        sample_predictions = self.model.predict(X_sample)
        
        if self.prediction_probabilities is not None:
            if len(self.X_test) > sample_size:
                sample_probs = self.prediction_probabilities[indices]
            else:
                sample_probs = self.prediction_probabilities
            pred_scores = np.max(sample_probs, axis=1)  # Use max probability as score
        else:
            pred_scores = sample_predictions.astype(float)
        
        violations = 0
        total_comparisons = 0
        violation_details = []
        
        # Compare similar individuals
        for i in range(len(scaled_features)):
            for j in range(i + 1, len(scaled_features)):
                # If individuals are similar in non-sensitive features
                if distances[i][j] <= distance_threshold:
                    # Check if predictions are similar
                    pred_diff = abs(pred_scores[i] - pred_scores[j])
                    
                    if pred_diff > prediction_threshold:
                        violations += 1
                        violation_details.append({
                            'individual_1': i,
                            'individual_2': j,
                            'feature_distance': distances[i][j],
                            'prediction_difference': pred_diff,
                            'prediction_1': float(pred_scores[i]),
                            'prediction_2': float(pred_scores[j])
                        })
                    
                    total_comparisons += 1
        
        fairness_rate = 1 - (violations / total_comparisons) if total_comparisons > 0 else 1
        individual_fairness_threshold = 0.9  # 90% fairness threshold
        
        result = {
            'violations': violations,
            'total_comparisons': total_comparisons,
            'fairness_rate': fairness_rate,
            'passes_threshold': fairness_rate >= individual_fairness_threshold,
            'violation_details': violation_details[:10],  # Store first 10 violations for analysis
            'fairness_metric': 'individual_fairness',
            'distance_threshold': distance_threshold,
            'prediction_threshold': prediction_threshold,
            'sample_size': len(scaled_features)
        }
        
        self.fairness_results['individual_fairness'] = result
        return result
    
    def test_calibration_across_groups(self, n_bins=10):
        """Test prediction calibration across different groups"""
        if self.prediction_probabilities is None:
            return {
                'error': 'Model does not support probability predictions',
                'fairness_metric': 'calibration'
            }
        
        results = {}
        pred_probs = self.prediction_probabilities[:, 1] if self.prediction_probabilities.shape[1] > 1 else self.prediction_probabilities[:, 0]
        
        for feature_name, feature_key in self.sensitive_features.items():
            if isinstance(feature_key, str):
                sensitive_values = self.X_test[feature_key] if isinstance(self.X_test, pd.DataFrame) else self.X_test[:, self.feature_names.index(feature_key)]
            else:
                sensitive_values = self.X_test.iloc[:, feature_key] if isinstance(self.X_test, pd.DataFrame) else self.X_test[:, feature_key]
            
            groups = np.unique(sensitive_values)
            group_calibration = {}
            
            for group in groups:
                group_mask = sensitive_values == group
                group_probs = pred_probs[group_mask]
                group_actual = self.y_test[group_mask]
                
                if len(group_probs) == 0:
                    continue
                
                # Create probability bins
                bin_boundaries = np.linspace(0, 1, n_bins + 1)
                bin_lowers = bin_boundaries[:-1]
                bin_uppers = bin_boundaries[1:]
                
                calibration_errors = []
                bin_details = []
                
                for bin_lower, bin_upper in zip(bin_lowers, bin_uppers):
                    bin_mask = (group_probs >= bin_lower) & (group_probs < bin_upper)
                    
                    if np.sum(bin_mask) > 0:
                        bin_probs = group_probs[bin_mask]
                        bin_actual = group_actual[bin_mask]
                        
                        predicted_prob = np.mean(bin_probs)
                        actual_prob = np.mean(bin_actual)
                        calibration_error = abs(predicted_prob - actual_prob)
                        
                        calibration_errors.append(calibration_error)
                        bin_details.append({
                            'bin_range': f'[{bin_lower:.2f}, {bin_upper:.2f})',
                            'count': int(np.sum(bin_mask)),
                            'predicted_prob': predicted_prob,
                            'actual_prob': actual_prob,
                            'calibration_error': calibration_error
                        })
                
                avg_calibration_error = np.mean(calibration_errors) if calibration_errors else 0
                max_calibration_error = np.max(calibration_errors) if calibration_errors else 0
                
                group_calibration[str(group)] = {
                    'average_calibration_error': avg_calibration_error,
                    'max_calibration_error': max_calibration_error,
                    'bin_details': bin_details,
                    'group_size': len(group_probs)
                }
            
            # Compare calibration across groups
            if len(group_calibration) >= 2:
                avg_errors = [cal['average_calibration_error'] for cal in group_calibration.values()]
                calibration_difference = max(avg_errors) - min(avg_errors)
                well_calibrated = max(avg_errors) < 0.1  # Good calibration threshold
            else:
                calibration_difference = 0
                well_calibrated = True
            
            results[feature_name] = {
                'group_calibration': group_calibration,
                'calibration_difference': calibration_difference,
                'well_calibrated': well_calibrated,
                'fairness_metric': 'calibration'
            }
        
        self.fairness_results['calibration'] = results
        return results
    
    def test_disparate_impact(self, threshold=0.8):
        """Test disparate impact (80% rule)"""
        results = {}
        
        for feature_name, feature_key in self.sensitive_features.items():
            if isinstance(feature_key, str):
                sensitive_values = self.X_test[feature_key] if isinstance(self.X_test, pd.DataFrame) else self.X_test[:, self.feature_names.index(feature_key)]
            else:
                sensitive_values = self.X_test.iloc[:, feature_key] if isinstance(self.X_test, pd.DataFrame) else self.X_test[:, feature_key]
            
            groups = np.unique(sensitive_values)
            group_rates = {}
            
            for group in groups:
                group_mask = sensitive_values == group
                group_predictions = self.predictions[group_mask]
                
                if len(group_predictions) > 0:
                    positive_rate = np.mean(group_predictions == 1)
                    group_rates[str(group)] = positive_rate
            
            if len(group_rates) >= 2:
                # Calculate disparate impact ratio
                rates = list(group_rates.values())
                max_rate = max(rates)
                min_rate = min(rates)
                
                disparate_impact_ratio = min_rate / max_rate if max_rate > 0 else 1
                
                # Find privileged and unprivileged groups
                privileged_group = max(group_rates.keys(), key=lambda k: group_rates[k])
                unprivileged_group = min(group_rates.keys(), key=lambda k: group_rates[k])
                
                passes_80_rule = disparate_impact_ratio >= threshold
            else:
                disparate_impact_ratio = 1
                privileged_group = unprivileged_group = list(group_rates.keys())[0] if group_rates else "Unknown"
                passes_80_rule = True
            
            results[feature_name] = {
                'group_rates': group_rates,
                'disparate_impact_ratio': disparate_impact_ratio,
                'privileged_group': privileged_group,
                'unprivileged_group': unprivileged_group,
                'passes_80_rule': passes_80_rule,
                'fairness_metric': 'disparate_impact',
                'threshold': threshold
            }
        
        self.fairness_results['disparate_impact'] = results
        return results
    
    def generate_fairness_report(self):
        """Generate comprehensive fairness assessment report"""
        
        # Count fairness violations
        total_tests = 0
        passed_tests = 0
        
        fairness_summary = {}
        
        for metric_name, metric_results in self.fairness_results.items():
            if metric_name == 'individual_fairness':
                total_tests += 1
                if metric_results.get('passes_threshold', False):
                    passed_tests += 1
                fairness_summary[metric_name] = {
                    'passed': metric_results.get('passes_threshold', False),
                    'score': metric_results.get('fairness_rate', 0)
                }
            else:
                for feature_name, feature_results in metric_results.items():
                    total_tests += 1
                    if feature_results.get('passes_threshold', False):
                        passed_tests += 1
                    
                    if metric_name not in fairness_summary:
                        fairness_summary[metric_name] = {}
                    
                    fairness_summary[metric_name][feature_name] = {
                        'passed': feature_results.get('passes_threshold', False),
                        'details': feature_results
                    }
        
        # Generate recommendations
        recommendations = self._generate_fairness_recommendations()
        
        # Calculate overall fairness score
        overall_fairness_score = passed_tests / total_tests if total_tests > 0 else 1
        
        report = {
            'fairness_assessment': {
                'summary': {
                    'total_tests': total_tests,
                    'passed_tests': passed_tests,
                    'overall_fairness_score': overall_fairness_score,
                    'fairness_grade': self._assign_fairness_grade(overall_fairness_score)
                },
                'detailed_results': self.fairness_results,
                'fairness_summary': fairness_summary,
                'recommendations': recommendations,
                'assessment_timestamp': datetime.now().isoformat()
            }
        }
        
        return report
    
    def _assign_fairness_grade(self, score):
        """Assign fairness grade based on score"""
        if score >= 0.95:
            return 'A+ (Excellent)'
        elif score >= 0.90:
            return 'A (Very Good)'
        elif score >= 0.80:
            return 'B (Good)'
        elif score >= 0.70:
            return 'C (Fair)'
        elif score >= 0.60:
            return 'D (Poor)'
        else:
            return 'F (Failing)'
    
    def _generate_fairness_recommendations(self):
        """Generate actionable fairness recommendations"""
        recommendations = []
        
        # Check demographic parity
        if 'demographic_parity' in self.fairness_results:
            for feature, results in self.fairness_results['demographic_parity'].items():
                if not results['passes_threshold']:
                    max_diff = results['max_difference']
                    recommendations.append(
                        f"Demographic parity violation in {feature}: {max_diff:.3f} difference in positive prediction rates"
                    )
                    recommendations.append(
                        f"Consider data rebalancing or bias mitigation techniques for {feature}"
                    )
        
        # Check equalized odds
        if 'equalized_odds' in self.fairness_results:
            for feature, results in self.fairness_results['equalized_odds'].items():
                if not results['passes_threshold']:
                    tpr_diff = results['tpr_difference']
                    fpr_diff = results['fpr_difference']
                    recommendations.append(
                        f"Equalized odds violation in {feature}: TPR diff={tpr_diff:.3f}, FPR diff={fpr_diff:.3f}"
                    )
                    recommendations.append(
                        f"Consider post-processing techniques to equalize error rates across {feature} groups"
                    )
        
        # Check individual fairness
        if 'individual_fairness' in self.fairness_results:
            if not self.fairness_results['individual_fairness'].get('passes_threshold', True):
                fairness_rate = self.fairness_results['individual_fairness']['fairness_rate']
                recommendations.append(
                    f"Individual fairness violation: {fairness_rate:.2%} fairness rate"
                )
                recommendations.append(
                    "Consider using fairness-aware algorithms or adding fairness constraints during training"
                )
        
        # Check calibration
        if 'calibration' in self.fairness_results:
            for feature, results in self.fairness_results['calibration'].items():
                if not results.get('well_calibrated', True):
                    recommendations.append(
                        f"Poor calibration across {feature} groups"
                    )
                    recommendations.append(
                        f"Consider calibration techniques like Platt scaling or isotonic regression"
                    )
        
        # Check disparate impact
        if 'disparate_impact' in self.fairness_results:
            for feature, results in self.fairness_results['disparate_impact'].items():
                if not results['passes_80_rule']:
                    ratio = results['disparate_impact_ratio']
                    recommendations.append(
                        f"Disparate impact in {feature}: {ratio:.2%} ratio (below 80% rule)"
                    )
                    recommendations.append(
                        f"Consider data preprocessing or algorithmic interventions for {feature}"
                    )
        
        if not recommendations:
            recommendations.append("Model demonstrates good fairness across all tested metrics")
            recommendations.append("Continue monitoring fairness in production deployment")
        else:
            recommendations.append("Implement bias mitigation strategies before production deployment")
            recommendations.append("Establish ongoing fairness monitoring and alerting systems")
        
        return recommendations

# Usage example
def run_comprehensive_fairness_testing():
    """Example of running comprehensive fairness testing"""
    # Load model and test data
    model = joblib.load('trained_model.pkl')
    X_test = pd.read_csv('test_features.csv')
    y_test = pd.read_csv('test_labels.csv')['target']
    
    # Define sensitive features
    sensitive_features = {
        'gender': 'gender',  # Column name
        'race': 'race',
        'age_group': 'age_group'
    }
    
    # Initialize fairness test suite
    fairness_tester = ComprehensiveFairnessTestSuite(
        model, X_test, y_test, sensitive_features, 
        feature_names=X_test.columns.tolist()
    )
    
    # Run all fairness tests
    print("Running comprehensive fairness testing...")
    
    demographic_parity = fairness_tester.test_demographic_parity(threshold=0.1)
    equalized_odds = fairness_tester.test_equalized_odds(threshold=0.1)
    equality_opportunity = fairness_tester.test_equality_of_opportunity(threshold=0.1)
    individual_fairness = fairness_tester.test_individual_fairness()
    calibration = fairness_tester.test_calibration_across_groups()
    disparate_impact = fairness_tester.test_disparate_impact(threshold=0.8)
    
    # Generate comprehensive report
    fairness_report = fairness_tester.generate_fairness_report()
    
    # Print summary
    print("\n=== Fairness Assessment Results ===")
    summary = fairness_report['fairness_assessment']['summary']
    print(f"Overall Fairness Score: {summary['overall_fairness_score']:.2%}")
    print(f"Fairness Grade: {summary['fairness_grade']}")
    print(f"Tests Passed: {summary['passed_tests']}/{summary['total_tests']}")
    
    print("\n=== Recommendations ===")
    for rec in fairness_report['fairness_assessment']['recommendations']:
        print(f"- {rec}")
    
    # Save detailed report
    with open('fairness_assessment_report.json', 'w') as f:
        import json
        json.dump(fairness_report, f, indent=2)
    
    return fairness_report
```

## Resume Showcase Tips

### Strong AI/ML Testing Experience
```
✅ Excellent Examples:
• "Developed comprehensive ML model testing framework detecting 95% of bias issues before production deployment"
• "Implemented automated model drift detection system reducing false positive rates by 40% and preventing 12 model failures"
• "Created advanced bias testing suite ensuring fairness across 8 demographic groups, achieving 92% fairness score"
• "Built adversarial testing framework identifying 25+ model vulnerabilities and improving robustness by 60%"
• "Led AI ethics testing initiative establishing company-wide responsible AI practices and compliance standards"

✅ Technical Skills to Highlight:
• Model validation techniques: A/B testing, cross-validation, statistical significance testing
• Bias detection frameworks: Demographic parity, equalized odds, individual fairness, disparate impact
• Drift detection systems: Data drift, concept drift, model drift monitoring and alerting
• Security testing: Adversarial attacks, prompt injection, data poisoning, robustness testing
• ML tools: MLflow, Weights & Biases, Great Expectations, Fairlearn, Alibi, TensorFlow Extended
• Statistical analysis: Hypothesis testing, distribution analysis, causal inference
```

### Quantifiable Achievements
- Model accuracy improvements (% increase in precision, recall, F1-score)
- Bias reduction metrics (fairness score improvements, demographic parity achievements)
- Drift detection effectiveness (early warning accuracy, false positive reduction)
- Cost savings from preventing model failures ($ amount, incident reduction)
- Security vulnerabilities identified and mitigated (number of attacks prevented)
- Compliance achievements (regulatory audit success rates)

## Learning Path 📚

### Beginner (0-4 months)
1. **ML Fundamentals & Testing Basics**
   ```
   Core Concepts:
   - Supervised vs unsupervised learning algorithms
   - Model training, validation, and testing workflows
   - Performance metrics: accuracy, precision, recall, F1-score, AUC-ROC
   - Overfitting, underfitting, and generalization concepts
   - Cross-validation techniques and implementation
   ```

2. **Basic Model Testing Implementation**
   ```python
   # Simple model validation example
   from sklearn.model_selection import train_test_split, cross_val_score
   from sklearn.metrics import accuracy_score, classification_report
   
   # Basic model testing workflow
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
   model.fit(X_train, y_train)
   predictions = model.predict(X_test)
   
   # Evaluate performance
   accuracy = accuracy_score(y_test, predictions)
   cv_scores = cross_val_score(model, X, y, cv=5)
   print(f"Accuracy: {accuracy:.3f}, CV Score: {cv_scores.mean():.3f}")
   ```

3. **Data Quality Testing Introduction**
   - Missing value detection and handling
   - Outlier identification techniques
   - Data type validation and consistency checks
   - Distribution analysis and visualization

### Intermediate (4-8 months)
1. **Advanced Model Validation Techniques**
   ```python
   # Advanced validation with statistical testing
   from scipy import stats
   import numpy as np
   
   def statistical_model_comparison(model1_scores, model2_scores):
       # Paired t-test for model comparison
       statistic, p_value = stats.ttest_rel(model1_scores, model2_scores)
       
       # Effect size calculation
       effect_size = (np.mean(model1_scores) - np.mean(model2_scores)) / np.std(model1_scores - model2_scores)
       
       return {
           'statistic': statistic,
           'p_value': p_value,
           'effect_size': effect_size,
           'significant_difference': p_value < 0.05
       }
   ```

2. **Bias and Fairness Testing Deep Dive**
   - Understanding different types of bias (selection, confirmation, algorithmic)
   - Implementing fairness metrics (demographic parity, equalized odds)
   - Protected attribute identification and handling
   - Bias mitigation techniques and evaluation

3. **Model Drift Detection Systems**
   - Statistical drift tests (KS test, chi-square, PSI)
   - Continuous monitoring setup
   - Alert system implementation
   - Automated retraining trigger mechanisms

### Advanced (8+ months)
1. **Production ML Testing & MLOps**
   - A/B testing frameworks for ML models
   - Shadow mode testing implementation
   - Canary deployment strategies
   - Real-time model monitoring and observability

2. **AI Security and Robustness Testing**
   ```python
   # Adversarial testing example
   import numpy as np
   
   def generate_adversarial_examples(model, X, epsilon=0.1):
       """Generate adversarial examples using FGSM"""
       # Get model gradients
       with tf.GradientTape() as tape:
           tape.watch(X)
           predictions = model(X)
           loss = tf.keras.losses.sparse_categorical_crossentropy(y_true, predictions)
       
       # Calculate gradients
       gradients = tape.gradient(loss, X)
       
       # Generate adversarial examples
       adversarial_X = X + epsilon * tf.sign(gradients)
       
       return adversarial_X
   ```

3. **Advanced AI Ethics and Explainability**
   - Model interpretability techniques (SHAP, LIME, permutation importance)
   - Causal inference and counterfactual analysis
   - Privacy-preserving ML testing
   - Regulatory compliance testing (GDPR, CCPA, AI Act)

## Best Practices

### Comprehensive Testing Strategy
```python
# ML testing strategy implementation
class MLTestingStrategy:
    def __init__(self):
        self.test_pyramid = {
            'unit_tests': 60,      # 60% unit/component tests
            'integration_tests': 25, # 25% integration tests
            'system_tests': 15     # 15% end-to-end system tests
        }
        
        self.testing_phases = [
            'data_validation',
            'model_validation', 
            'bias_fairness_testing',
            'robustness_testing',
            'performance_testing',
            'drift_monitoring'
        ]
    
    def execute_testing_pipeline(self, model, data):
        """Execute comprehensive ML testing pipeline"""
        results = {}
        
        for phase in self.testing_phases:
            phase_results = self.run_testing_phase(phase, model, data)
            results[phase] = phase_results
            
            # Stop if critical issues found
            if phase_results.get('critical_issues', False):
                results['pipeline_status'] = 'FAILED'
                break
        
        return results
```

### Continuous ML Testing Integration
```yaml
# GitHub Actions workflow for ML testing
name: ML Model Testing Pipeline

on:
  push:
    branches: [ main, develop ]
    paths: [ 'models/**', 'data/**', 'src/**' ]

jobs:
  ml-testing:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.9'
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install pytest coverage great-expectations
    
    - name: Run data validation tests
      run: |
        python -m pytest tests/data_validation/ -v
        great_expectations checkpoint run data_quality_checkpoint
    
    - name: Run model validation tests
      run: |
        python -m pytest tests/model_validation/ -v --cov=src/models
    
    - name: Run bias and fairness tests
      run: |
        python tests/fairness_testing.py
    
    - name: Run drift detection tests
      run: |
        python tests/drift_detection.py
    
    - name: Generate ML testing report
      run: |
        python scripts/generate_ml_report.py
    
    - name: Upload test artifacts
      uses: actions/upload-artifact@v3
      with:
        name: ml-test-results
        path: |
          test_results/
          reports/
          coverage.xml
```

### Model Monitoring and Alerting
- Set up real-time performance monitoring dashboards
- Implement automated drift detection with appropriate thresholds
- Create escalation procedures for different types of model failures
- Establish model rollback procedures for critical issues
- Maintain model versioning and experiment tracking

### Documentation and Compliance
```python
def generate_model_card(model_info, test_results):
    """Generate comprehensive model card for transparency"""
    model_card = {
        'model_details': {
            'name': model_info['name'],
            'version': model_info['version'],
            'type': model_info['type'],
            'architecture': model_info['architecture'],
            'training_date': model_info['training_date']
        },
        'intended_use': {
            'primary_uses': model_info['intended_uses'],
            'out_of_scope_uses': model_info['limitations']
        },
        'performance_metrics': test_results['performance'],
        'fairness_assessment': test_results['fairness'],
        'bias_evaluation': test_results['bias_analysis'],
        'ethical_considerations': model_info['ethical_notes'],
        'limitations': model_info['known_limitations'],
        'recommendations': test_results['recommendations']
    }
    
    return model_card
```

---

**Next Step**: Integrate with development workflows! Continue to [CI/CD & DevOps Testing](./08-cicd-devops-testing.md) to learn GitHub Actions, Docker, and deployment testing strategies.
