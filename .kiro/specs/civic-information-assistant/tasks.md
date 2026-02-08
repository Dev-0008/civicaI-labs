# Implementation Plan: Civic Information Assistant

## Overview

This implementation plan breaks down the civic information assistant into incremental coding tasks. The approach follows a bottom-up strategy: building core data models and utilities first, then services, and finally the conversation controller that orchestrates everything. Each task includes property-based tests to validate correctness properties from the design document.

The implementation uses Python with a focus on modularity, testability, and low-bandwidth operation.

## Tasks

- [ ] 1. Set up project structure and dependencies
  - Create Python project with virtual environment
  - Install dependencies: pytest, hypothesis (for property-based testing), spacy (for NLP)
  - Set up directory structure: `src/`, `tests/`, `data/`
  - Create `requirements.txt` with all dependencies
  - Set up pytest configuration for property-based testing (minimum 100 iterations)
  - _Requirements: All_

- [ ] 2. Implement core data models
  - [ ] 2.1 Create data model classes
    - Implement `UserProfile` class with fields: userId, age, income, occupation, languagePreference, timestamps
    - Implement `SchemeInfo` class with all fields from design
    - Implement `EligibilityCriteria` class with age/income bounds and occupation list
    - Implement `EligibilityResult` class with eligibility status and explanations
    - Implement `Document` and `Step` classes for application guidance
    - Implement `SessionState` and `Message` classes for conversation management
    - Implement `Response` and `ErrorResponse` classes
    - Add validation methods to each class
    - _Requirements: 2.1, 2.2, 2.3, 4.1, 5.1, 5.2_
  
  - [ ]* 2.2 Write property test for profile validation
    - **Property 4: Age Validation Accepts Valid Range**
    - **Property 5: Income Validation Accepts Non-Negative**
    - **Validates: Requirements 2.4, 2.5**

- [ ] 3. Implement user storage and profile manager
  - [ ] 3.1 Create UserStorage class
    - Implement file-based storage using JSON
    - Implement `save()`, `load()`, `delete()`, `exists()` methods
    - Add encryption for sensitive data (age, income)
    - Create storage directory if not exists
    - _Requirements: 9.2, 9.4_
  
  - [ ] 3.2 Create ProfileManager class
    - Implement `createProfile()`, `getProfile()`, `updateProfile()` methods
    - Implement `validateField()` with validation rules for age, income, occupation
    - Implement `isProfileComplete()` to check if all required fields are set
    - Integrate with UserStorage for persistence
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7_
  
  - [ ]* 3.3 Write property tests for profile manager
    - **Property 6: Profile Update Round Trip**
    - **Property 24: Profile Deletion Completeness**
    - **Validates: Requirements 2.6, 2.7, 9.4**

- [ ] 4. Implement scheme database
  - [ ] 4.1 Create SchemeDatabase class
    - Implement JSON-based scheme storage
    - Implement `getSchemeById()`, `getSchemeByName()`, `searchSchemes()` methods
    - Implement `getAllSchemes()` and `filterByCategory()` methods
    - Create sample scheme data with various eligibility criteria
    - _Requirements: 3.1, 3.5_
  
  - [ ] 4.2 Create sample scheme data
    - Create at least 5 sample schemes with different eligibility criteria
    - Include schemes with age bounds, income bounds, occupation requirements
    - Include schemes with multiple application methods (online/offline)
    - Store in `data/schemes.json`
    - _Requirements: 3.1, 3.5_
  
  - [ ]* 4.3 Write property test for scheme retrieval
    - **Property 7: Scheme Request Returns Document**
    - **Validates: Requirements 3.1**

- [ ] 5. Implement NLP engine
  - [ ] 5.1 Create intent classifier
    - Implement `IntentClassifier` class with keyword-based classification
    - Create keyword dictionaries for each intent type (SCHEME_INFO, ELIGIBILITY_CHECK, APPLICATION_GUIDANCE, PROFILE_UPDATE)
    - Implement `detectIntent()` method with confidence scoring
    - Implement `getConfidenceScore()` method
    - Set confidence threshold (e.g., 0.6)
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_
  
  - [ ] 5.2 Create entity extractor
    - Implement `EntityExtractor` class
    - Implement regex-based extraction for numbers (age, income)
    - Implement scheme name matching using database lookup
    - Implement `extractEntities()` method returning list of entities
    - _Requirements: 1.1_
  
  - [ ] 5.3 Create NLPEngine class
    - Integrate IntentClassifier and EntityExtractor
    - Implement main `detectIntent()` and `extractEntities()` methods
    - Handle errors gracefully with fallback to UNCLEAR intent
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_
  
  - [ ]* 5.4 Write property tests for NLP engine
    - **Property 1: Intent Detection Returns Valid Intent**
    - **Property 2: Keyword-Based Intent Classification**
    - **Property 3: Low Confidence Triggers Clarification**
    - **Validates: Requirements 1.1, 1.2, 1.3, 1.4, 1.5**

- [ ] 6. Checkpoint - Ensure core components work
  - Run all tests to verify data models, storage, and NLP engine
  - Ensure all tests pass, ask the user if questions arise

- [ ] 7. Implement eligibility matcher
  - [ ] 7.1 Create EligibilityMatcher class
    - Implement `match()` method comparing profile against criteria
    - Implement age bounds checking: `minAge <= age <= maxAge`
    - Implement income bounds checking: `minIncome <= income <= maxIncome`
    - Implement occupation matching: `occupation in allowed_occupations`
    - Implement `explainMismatch()` to generate explanation text
    - Return EligibilityResult with matched/unmatched criteria
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6_
  
  - [ ]* 7.2 Write property tests for eligibility matcher
    - **Property 10: Eligibility Matching Performs Comparison**
    - **Property 11: Eligibility Determination Correctness**
    - **Property 12: Ineligibility Explanation Completeness**
    - **Property 13: Missing Information Triggers Request**
    - **Validates: Requirements 4.1, 4.2, 4.3, 4.4, 4.5**

- [ ] 8. Implement text summarizer
  - [ ] 8.1 Create TextSummarizer class
    - Implement `simplify()` method with rule-based simplification
    - Create dictionary of complex-to-simple term mappings
    - Implement sentence breaking for long sentences (> 20 words)
    - Implement vocabulary replacement using dictionary
    - Implement `extractKeyPoints()` to identify main points
    - Handle errors by returning original text
    - _Requirements: 3.2, 3.3_
  
  - [ ]* 8.2 Write property tests for text summarizer
    - **Property 8: Document Summarization Always Produces Output**
    - **Property 27: Summarizer Failure Fallback**
    - **Validates: Requirements 3.2, 10.3**

- [ ] 9. Implement translation service
  - [ ] 9.1 Create TranslationService class
    - Implement `translate()` method (initially with mock translations)
    - Implement `isLanguageSupported()` checking against supported language list
    - Implement `getSupportedLanguages()` returning list of supported languages
    - Implement fallback to English when translation unavailable
    - Add caching for common translations
    - _Requirements: 6.1, 6.2, 6.3, 6.4_
  
  - [ ]* 9.2 Write property tests for translation service
    - **Property 17: Language Preference Round Trip**
    - **Property 18: Output Translation to Preferred Language**
    - **Property 19: Unsupported Language Fallback**
    - **Validates: Requirements 6.1, 6.2, 6.3**

- [ ] 10. Implement scheme service
  - [ ] 10.1 Create SchemeService class
    - Implement `getSchemeInfo()` integrating database and summarizer
    - Implement `checkEligibility()` integrating eligibility matcher
    - Implement `getApplicationSteps()` retrieving steps from scheme data
    - Implement `getRequiredDocuments()` retrieving documents from scheme data
    - Implement `searchSchemes()` for keyword-based search
    - Implement `getEligibleSchemes()` filtering schemes by profile
    - Handle errors gracefully (scheme not found, data unavailable)
    - _Requirements: 3.1, 3.2, 3.3, 4.1, 4.2, 4.3, 4.4, 4.5, 5.1, 5.2_
  
  - [ ]* 10.2 Write property tests for scheme service
    - **Property 9: Scheme Response Contains Simplified Text**
    - **Property 14: Application Request Returns Documents**
    - **Property 15: Application Steps Sequential Ordering**
    - **Property 16: Multiple Application Methods Displayed**
    - **Property 26: Scheme Data Unavailable Handling**
    - **Validates: Requirements 3.3, 5.1, 5.2, 5.3, 5.4, 10.2**

- [ ] 11. Implement output formatter
  - [ ] 11.1 Create OutputFormatter class
    - Implement `formatSchemeInfo()` creating readable scheme descriptions
    - Implement `formatEligibilityResult()` formatting eligibility responses
    - Implement `formatApplicationSteps()` with numbered list
    - Implement `formatDocumentList()` with bullet points
    - Implement `addDisclaimer()` appending disclaimer text to responses
    - Ensure all formatted output is plain text
    - Implement payload size checking (< 50KB)
    - _Requirements: 7.1, 8.2_
  
  - [ ]* 11.2 Write property tests for output formatter
    - **Property 20: Response Payload Size Limit**
    - **Property 22: Scheme Response Includes Disclaimer**
    - **Validates: Requirements 7.1, 8.2**

- [ ] 12. Checkpoint - Ensure all services work
  - Run all tests to verify eligibility, summarization, translation, and formatting
  - Ensure all tests pass, ask the user if questions arise

- [ ] 13. Implement conversation controller
  - [ ] 13.1 Create ConversationController class
    - Implement `initializeSession()` creating new session state
    - Implement `getSessionState()` and `updateSessionState()` for state management
    - Implement `handleUserInput()` as main entry point
    - Implement conversation flow logic:
      - Check if profile is complete, if not collect profile data
      - Use NLP engine to detect intent
      - Route to appropriate service based on intent
      - Format response using output formatter
      - Translate response if language preference set
    - Implement profile collection flow with validation
    - Handle low confidence by requesting clarification
    - _Requirements: 1.1, 1.5, 2.1, 2.2, 2.3, 6.2, 9.1_
  
  - [ ] 13.2 Implement error handling in controller
    - Catch NLP engine errors and display user-friendly messages
    - Catch scheme service errors and suggest retry
    - Catch unexpected errors and log them
    - Implement error response formatting
    - _Requirements: 10.1, 10.2, 10.5_
  
  - [ ]* 13.3 Write property tests for conversation controller
    - **Property 23: Privacy Notice During Collection**
    - **Property 25: NLP Failure Error Handling**
    - **Property 28: Unexpected Error Handling**
    - **Validates: Requirements 9.1, 10.1, 10.5**

- [ ] 14. Implement disclaimer management
  - [ ] 14.1 Create disclaimer text and display logic
    - Create disclaimer text file with required statements
    - Implement disclaimer display on first access
    - Ensure disclaimer states: informational support only, not legal advice, verify with official sources
    - Integrate disclaimer into output formatter
    - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5_
  
  - [ ] 14.2 Write unit tests for disclaimer
    - Test disclaimer displays on first access
    - Test disclaimer contains all required text elements
    - Test disclaimer included in scheme responses
    - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5_

- [ ] 15. Implement command-line interface
  - [ ] 15.1 Create CLI application
    - Implement main loop for user interaction
    - Display welcome message and disclaimer on startup
    - Prompt for user input and display responses
    - Support commands: /help, /profile, /language, /quit
    - Handle keyboard interrupts gracefully
    - _Requirements: All_
  
  - [ ] 15.2 Add language selection to CLI
    - Display available languages on startup
    - Allow user to select language preference
    - Update profile with language preference
    - _Requirements: 6.1, 6.2, 6.3_

- [ ] 16. Integration and end-to-end testing
  - [ ]* 16.1 Write integration tests
    - Test complete conversation flow: greeting → profile collection → scheme inquiry → eligibility check → application guidance
    - Test profile update flow: create profile → update field → verify changes
    - Test language switching: set language → verify responses translated
    - Test error recovery: trigger error → verify handling → continue conversation
    - _Requirements: All_

- [ ] 17. Create documentation
  - [ ] 17.1 Write README.md
    - Document installation instructions
    - Document how to run the application
    - Document available commands
    - Document how to add new schemes
    - Include disclaimer and privacy information
    - _Requirements: All_
  
  - [ ] 17.2 Write API documentation
    - Document all public classes and methods
    - Include usage examples
    - Document data models and their fields
    - _Requirements: All_

- [ ] 18. Final checkpoint - Complete system validation
  - Run all unit tests and property tests
  - Run integration tests
  - Test with sample conversations
  - Verify all 28 correctness properties pass
  - Ensure all tests pass, ask the user if questions arise

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Property tests use Hypothesis library with minimum 100 iterations
- All property tests are tagged with format: `# Feature: civic-information-assistant, Property N: [property text]`
- Focus on text-based interactions for low-bandwidth operation
- Implement encryption for user data storage
- Use JSON for data persistence (schemes and user profiles)
- Mock translation service initially, can integrate real API later
- Sample scheme data should cover diverse eligibility criteria for testing
