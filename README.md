# Inclusive-Classroom-Assessment
This large-scale, object-oriented system is designed to evaluate student code submissions using inclusive design principles. 
import abc
import json
import re
from typing import Dict, List, Any, Optional

# =====================================================================
# 1. CORE DATA STRUCTURES & ACCESSIBILITY MODELS
# =====================================================================

class AccommodationProfile:
    """Manages universal design accommodations for students."""
    def __init__(self, extended_time: bool = False, simplified_feedback: bool = False, 
                 allow_partial_pseudocode: bool = False, alternative_input_format: bool = False):
        self.extended_time = extended_time
        self.simplified_feedback = simplified_feedback
        self.allow_partial_pseudocode = allow_partial_pseudocode
        self.alternative_input_format = alternative_input_format

    def to_dict(self) -> Dict[str, bool]:
        return self.__dict__


class Student:
    """Represents a student enrolled in the inclusive course ecosystem."""
    def __init__(self, student_id: str, name: str, profile: Optional[AccommodationProfile] = None):
        self.student_id = student_id
        self.name = name
        self.profile = profile if profile else AccommodationProfile()
        self.submissions: Dict[str, Any] = {}

    def add_submission(self, assignment_id: str, payload: Dict[str, Any]):
        self.submissions[assignment_id] = payload


# =====================================================================
# 2. ASSESSMENT ENGINE & INCLUSIVE CRITERIA (UNIVERSAL DESIGN)
# =====================================================================

class AssessmentCriterion(abc.ABC):
    """Abstract Base Class establishing inclusive assessment benchmarks."""
    @abc.abstractmethod
    def evaluate(self, code: str, conceptual_explanation: str, profile: AccommodationProfile) -> Dict[str, Any]:
        pass


class SyntaxAndLogicCriterion(AssessmentCriterion):
    """Evaluates code correctness flexibly, balancing logic vs strict syntax rules."""
    def __init__(self, target_keywords: List[str], required_constructs: List[str]):
        self.target_keywords = target_keywords
        self.required_constructs = required_constructs

    def evaluate(self, code: str, conceptual_explanation: str, profile: AccommodationProfile) -> Dict[str, Any]:
        score = 0.0
        max_score = 40.0
        feedback = []
        
        # Clean up input variations
        clean_code = re.sub(r'\s+', ' ', code).lower()
        
        # Check required algorithmic keywords
        keyword_matches = [kw for kw in self.target_keywords if kw.lower() in clean_code]
        if len(keyword_matches) == len(self.target_keywords):
            score += 20.0
            feedback.append("Excellent! All critical logic pathways are implemented.")
        elif len(keyword_matches) > 0:
            score += (len(keyword_matches) / len(self.target_keywords)) * 20.0
            feedback.append(f"Partial implementation found. Met keywords: {', '.join(keyword_matches)}")
        else:
            # Inclusive Fallback: Read the textual explanation if code syntax is broken
            if profile.allow_partial_pseudocode and len(conceptual_explanation) > 30:
                score += 10.0
                feedback.append("Syntax missing, but conceptual strategy validated through text explanation.")
            else:
                feedback.append("Core keywords missing. Review the programmatic logic guidelines.")

        # Structure analysis (e.g., loops, branching conditions)
        found_constructs = []
        for construct in self.required_constructs:
            if construct == "loop" and ("for " in code or "while " in code):
                found_constructs.append(construct)
            elif construct == "conditional" and ("if " in code or "elif " in code or "else:" in code):
                found_constructs.append(construct)

        if len(found_constructs) == len(self.required_constructs):
            score += 20.0
            feedback.append("Required control structures correctly structured.")
        else:
            score += (len(found_constructs) / len(self.required_constructs)) * 20.0
            feedback.append("Some expected structural blocks are missing or modified.")

        return {"score": round(score, 2), "max_score": max_score, "feedback": feedback}


class ExplanatoryCompetencyCriterion(AssessmentCriterion):
    """Provides equitable assessment pathways for neurodiverse learners via textual verification."""
    def evaluate(self, code: str, conceptual_explanation: str, profile: AccommodationProfile) -> Dict[str, Any]:
        score = 0.0
        max_score = 30.0
        feedback = []

        word_count = len(conceptual_explanation.split())
        
        if word_count >= 25:
            score += 20.0
            feedback.append("Comprehensive self-reflection and structural intent submitted.")
        elif word_count >= 10:
            score += 10.0
            feedback.append("Brief concept overview received.")
        else:
            feedback.append("Please elaborate on your programming approach in the reflection text box.")

        # Keywords linking intent to computational design
        tokens = ["intent", "logic", "variable", "loop", "fix", "output", "error"]
        matched_tokens = [t for t in tokens if t in conceptual_explanation.lower()]
        
        if len(matched_tokens) >= 2:
            score += 10.0
            feedback.append("Strong semantic correlation between coding application and intent.")
        elif profile.simplified_feedback:
            score += 5.0  # Equitable compensation for simplified language profiles
            feedback.append("Concept alignment confirmed.")

        return {"score": round(score, 2), "max_score": max_score, "feedback": feedback}


# =====================================================================
# 3. REPORTING ENGINE & FEEDBACK MODIFIER
# =====================================================================

class InclusiveAssessmentSystem:
    """Coordinates assessments and modifies outputs based on equity guidelines."""
    def __init__(self, assignment_id: str, name: str):
        self.assignment_id = assignment_id
        self.name = name
        self.criteria: List[AssessmentCriterion] = []

    def register_criterion(self, criterion: AssessmentCriterion):
        self.criteria.append(criterion)

    def process_submission(self, student: Student, code: str, explanation: str) -> Dict[str, Any]:
        total_score = 0.0
        total_possible = 0.0
        compiled_feedbacks = []
        profile = student.profile

        for criterion in self.criteria:
            result = criterion.evaluate(code, explanation, profile)
            total_score += result["score"]
            total_possible += result["max_score"]
            compiled_feedbacks.extend(result["feedback"])

        # Inclusive Multi-Tier Adaptive Feedback Engine
        optimized_feedback = self._transform_feedback(compiled_feedbacks, profile)
        pass_threshold = total_possible * 0.50
        passed = total_score >= pass_threshold

        report_card = {
            "assignment_id": self.assignment_id,
            "assignment_name": self.name,
            "student_id": student.student_id,
            "student_name": student.name,
            "raw_score": total_score,
            "max_possible": total_possible,
            "percentage": round((total_score / total_possible) * 100, 2),
            "passed": passed,
            "accommodations_applied": profile.to_dict(),
            "actionable_feedback": optimized_feedback
        }
        
        student.add_submission(self.assignment_id, report_card)
        return report_card

    def _transform_feedback(self, feedbacks: List[str], profile: AccommodationProfile) -> List[str]:
        """Translates dense compiler-like outputs into plain, non-punitive, and actionable items."""
        if not profile.simplified_feedback:
            return feedbacks
        
        # Condense and clarify language targets for cognitive load reduction
        simplified = []
        for text in feedbacks:
            if "missing" in text.lower() or "broken" in text.lower():
                simplified.append("💡 Action item: Try reviewing your structure guidelines for missing helper pieces.")
            elif "excellent" in text.lower() or "correctly" in text.lower():
                simplified.append("🌟 Great job! Your logic approach works exactly as intended.")
            else:
                simplified.append(text)
        return list(set(simplified)) # Deduplicate for clarity


# =====================================================================
# 4. RUNTIME DEMONSTRATION & VERIFICATION
# =====================================================================

if __name__ == "__main__":
    print("--- Initializing Inclusive Assessment Platform Run ---\n")

    # 1. Establish an Assignment Blueprint
    assignment = InclusiveAssessmentSystem(assignment_id="LAB_01", name="Introductory Automation Script")
    
    # Register Evaluation Matrix Components 
    assignment.register_criterion(SyntaxAndLogicCriterion(
        target_keywords=["def", "return"], 
        required_constructs=["loop", "conditional"]
    ))
    assignment.register_criterion(ExplanatoryCompetencyCriterion())

    # 2. Setup Student Base Profiles (Diverse profiles to highlight inclusion strategies)
    student_a = Student(
        student_id="STU_991", 
        name="Alex Mercer", 
        profile=AccommodationProfile(allow_partial_pseudocode=True, simplified_feedback=True)
    )
    student_b = Student(
        student_id="STU_882", 
        name="Sam Vance", 
        profile=AccommodationProfile(allow_partial_pseudocode=False, simplified_feedback=False)
    )

    # 3. Process submissions
    # Submission from Alex: High narrative competency, but missed specific coding blocks
