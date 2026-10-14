# Issue #54: test evidence

Environment: macOS 26.5.2 arm64, Python 3.11.4, pytest 9.1.1, pypdf 6.20.1. Baseline `f89c06f`; fix `2254e535f18a8eff489430e5dd1c664830678a10`.

## Reproduction commands

The same script ran against an archive of `main` at `f89c06f` for before and the fix tree committed as `2254e53` for after. The baseline archive is separate from the fix checkout.

```python
from ingestion.parsers.resume_parser import ResumeParser

p = ResumeParser()
indented = """
    Experience:
    Senior Developer at TechCorp

    Education:
    BS Computer Science

    Skills: Python, JavaScript
"""
flush = "\n".join(line.strip() for line in indented.splitlines())
print("indented ->", p._detect_sections(indented))
print("flush    ->", p._detect_sections(flush))

r = ResumeParser()
res = r.parse('\n    John Smith\n    john@example.com\n\n    Education:\n    - B.S. Computer Science\n\n    Skills: Python\n')
print("issue-body example ->", res.metadata["detected_sections"])
```

### Before: original reproduction

```text
$ python -c "<script above>" (main archive)
indented -> []
flush    -> ['Education', 'Skills', 'Experience']
issue-body example -> []
```

### After: original reproduction

Section order is unspecified because the parser returns `list(set(detected))`. The meaningful comparison is the set of section names.

```text
$ python -c "<same script above>" (fix branch)
indented -> ['Experience', 'Skills', 'Education']
flush    -> ['Experience', 'Skills', 'Education']
issue-body example -> ['Skills', 'Education']
```

### Before: original failing test

```text
$ python -m pytest tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections --runxfail -v (main archive)
============================= test session starts ==============================
platform darwin -- Python 3.11.4, pytest-9.1.1, pluggy-1.6.0 -- /Users/rupeshvarma/.codex/.chatgpt-projects/g-p-6ab54d8209dc819183d574683d512174/codepath-pathreview/.venv/bin/python
cachedir: .pytest_cache
hypothesis profile 'default'
benchmark: 5.3.0 (defaults: timer=time.perf_counter disable_gc=False min_rounds=5 min_time=0.000005 max_time=1.0 calibration_precision=10 warmup=False warmup_iterations=100000)
rootdir: /Users/rupeshvarma/.codex/.chatgpt-projects/g-p-6ab54d8209dc819183d574683d512174/codepath-unit4/baseline-main
configfile: pyproject.toml
plugins: pytest_httpserver-1.2.0, platformdirs-4.13.0, cov-7.1.0, hypothesis-6.169.0, asyncio-1.4.0, benchmark-5.3.0, anyio-4.15.1
asyncio: mode=Mode.STRICT, debug=False, asyncio_default_fixture_loop_scope=None, asyncio_default_test_loop_scope=function
collecting ... collected 1 item

tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections FAILED [100%]

=================================== FAILURES ===================================
____________________ TestResumeParser.test_detect_sections _____________________

self = <tests.unit.test_resume_parser.TestResumeParser object at 0x106fc4090>
parser = <ingestion.parsers.resume_parser.ResumeParser object at 0x106fc5b90>

    @pytest.mark.xfail(
        strict=True, reason="issue #54: resume section detection fails on leading whitespace"
    )
    def test_detect_sections(self, parser):
        """Test section detection in resume text."""
        text = """
        Experience:
        Senior Developer at TechCorp
    
        Education:
        BS Computer Science
    
        Skills: Python, JavaScript
        """
        sections = parser._detect_sections(text)
    
        assert isinstance(sections, list)
>       assert len(sections) > 0
E       assert 0 > 0
E        +  where 0 = len([])

tests/unit/test_resume_parser.py:152: AssertionError
=========================== short test summary info ============================
FAILED tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections
============================== 1 failed in 0.16s ===============================
```

### After: parser tests

```text
$ python -m pytest tests/unit/test_resume_parser.py -v (fix branch)
============================= test session starts ==============================
platform darwin -- Python 3.11.4, pytest-9.1.1, pluggy-1.6.0 -- /Users/rupeshvarma/.codex/.chatgpt-projects/g-p-6ab54d8209dc819183d574683d512174/codepath-pathreview/.venv/bin/python
cachedir: .pytest_cache
hypothesis profile 'default'
benchmark: 5.3.0 (defaults: timer=time.perf_counter disable_gc=False min_rounds=5 min_time=0.000005 max_time=1.0 calibration_precision=10 warmup=False warmup_iterations=100000)
rootdir: /Users/rupeshvarma/.codex/.chatgpt-projects/g-p-6ab54d8209dc819183d574683d512174/codepath-pathreview
configfile: pyproject.toml
plugins: pytest_httpserver-1.2.0, platformdirs-4.13.0, cov-7.1.0, hypothesis-6.169.0, asyncio-1.4.0, benchmark-5.3.0, anyio-4.15.1
asyncio: mode=Mode.STRICT, debug=False, asyncio_default_fixture_loop_scope=None, asyncio_default_test_loop_scope=function
collecting ... collected 12 items

tests/unit/test_resume_parser.py::TestResumeParser::test_parse_single_column_resume_text PASSED [  8%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_resume_no_work_experience PASSED [ 16%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_multipage_pdf PASSED [ 25%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_markdown_resume PASSED [ 33%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_invalid_content_type PASSED [ 41%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_invalid_list_content PASSED [ 50%]
tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections PASSED [ 58%]
tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections_matches_unindented_text[    ] PASSED [ 66%]
tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections_matches_unindented_text[\t] PASSED [ 75%]
tests/unit/test_resume_parser.py::TestResumeParser::test_strip_markdown_syntax PASSED [ 83%]
tests/unit/test_resume_parser.py::TestResumeParser::test_pdf_parsing_error_handling PASSED [ 91%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_preserves_text_content PASSED [100%]

============================== 12 passed in 0.12s ==============================
```

### New regression test against baseline

The new comparison test was copied into a scratch `main` archive without changing baseline production code. Both parameter cases fail before the fix; both pass on the branch.

```text
$ python -m pytest tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections_matches_unindented_text -v (main archive, with the new test copied in)
============================= test session starts ==============================
platform darwin -- Python 3.11.4, pytest-9.1.1, pluggy-1.6.0 -- /Users/rupeshvarma/.codex/.chatgpt-projects/g-p-6ab54d8209dc819183d574683d512174/codepath-pathreview/.venv/bin/python
cachedir: .pytest_cache
hypothesis profile 'default'
benchmark: 5.3.0 (defaults: timer=time.perf_counter disable_gc=False min_rounds=5 min_time=0.000005 max_time=1.0 calibration_precision=10 warmup=False warmup_iterations=100000)
rootdir: /Users/rupeshvarma/.codex/.chatgpt-projects/g-p-6ab54d8209dc819183d574683d512174/codepath-unit4/baseline-main
configfile: pyproject.toml
plugins: pytest_httpserver-1.2.0, platformdirs-4.13.0, cov-7.1.0, hypothesis-6.169.0, asyncio-1.4.0, benchmark-5.3.0, anyio-4.15.1
asyncio: mode=Mode.STRICT, debug=False, asyncio_default_fixture_loop_scope=None, asyncio_default_test_loop_scope=function
collecting ... collected 2 items

codepath-unit4/baseline-main/tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections_matches_unindented_text[    ] FAILED [ 50%]
codepath-unit4/baseline-main/tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections_matches_unindented_text[\t] FAILED [100%]

=================================== FAILURES ===================================
_____ TestResumeParser.test_detect_sections_matches_unindented_text[    ] ______

self = <tests.unit.test_resume_parser.TestResumeParser object at 0x1076bded0>
parser = <ingestion.parsers.resume_parser.ResumeParser object at 0x1076bf4d0>
indent = '    '

    @pytest.mark.parametrize("indent", ["    ", "\t"])
    def test_detect_sections_matches_unindented_text(self, parser, indent):
        """Horizontal indentation preserves the detected section names."""
        text = (
            "Experience:\nSenior Developer at TechCorp\n\n"
            "Education:\nBS Computer Science\n\n"
            "Skills: Python, JavaScript\n"
        )
        indented = "\n".join(indent + line for line in text.splitlines())
    
        expected = {"Experience", "Education", "Skills"}
        assert set(parser._detect_sections(text)) == expected
>       assert set(parser._detect_sections(indented)) == expected
E       AssertionError: assert set() == {'Education',...ce', 'Skills'}
E         
E         Extra items in the right set:
E         'Skills'
E         'Experience'
E         'Education'
E         
E         Full diff:...
E         
E         ...Full output truncated (6 lines hidden), use '-vv' to show

codepath-unit4/baseline-main/tests/unit/test_resume_parser.py:158: AssertionError
______ TestResumeParser.test_detect_sections_matches_unindented_text[\t] _______

self = <tests.unit.test_resume_parser.TestResumeParser object at 0x1076be110>
parser = <ingestion.parsers.resume_parser.ResumeParser object at 0x1076bf090>
indent = '\t'

    @pytest.mark.parametrize("indent", ["    ", "\t"])
    def test_detect_sections_matches_unindented_text(self, parser, indent):
        """Horizontal indentation preserves the detected section names."""
        text = (
            "Experience:\nSenior Developer at TechCorp\n\n"
            "Education:\nBS Computer Science\n\n"
            "Skills: Python, JavaScript\n"
        )
        indented = "\n".join(indent + line for line in text.splitlines())
    
        expected = {"Experience", "Education", "Skills"}
        assert set(parser._detect_sections(text)) == expected
>       assert set(parser._detect_sections(indented)) == expected
E       AssertionError: assert set() == {'Education',...ce', 'Skills'}
E         
E         Extra items in the right set:
E         'Skills'
E         'Experience'
E         'Education'
E         
E         Full diff:...
E         
E         ...Full output truncated (6 lines hidden), use '-vv' to show

codepath-unit4/baseline-main/tests/unit/test_resume_parser.py:158: AssertionError
=========================== short test summary info ============================
FAILED codepath-unit4/baseline-main/tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections_matches_unindented_text[    ]
FAILED codepath-unit4/baseline-main/tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections_matches_unindented_text[\t]
============================== 2 failed in 0.32s ===============================
```

## Repository checks

All four commands below ran from the fix repository root using its `.venv`. Outputs are preserved, including warnings and the integration command’s nonzero result.

### Unit tests

Exit 0: 382 passed, 48 xfailed. The remaining xfails belong to other seeded issues; all five issue #54 markers were removed.

```text
$ make test-unit
.venv/bin/pytest tests/unit -v -m unit
============================= test session starts ==============================
platform darwin -- Python 3.11.4, pytest-9.1.1, pluggy-1.6.0 -- /Users/rupeshvarma/.codex/.chatgpt-projects/g-p-6ab54d8209dc819183d574683d512174/codepath-pathreview/.venv/bin/python
cachedir: .pytest_cache
hypothesis profile 'default'
benchmark: 5.3.0 (defaults: timer=time.perf_counter disable_gc=False min_rounds=5 min_time=0.000005 max_time=1.0 calibration_precision=10 warmup=False warmup_iterations=100000)
rootdir: /Users/rupeshvarma/.codex/.chatgpt-projects/g-p-6ab54d8209dc819183d574683d512174/codepath-pathreview
configfile: pyproject.toml
plugins: pytest_httpserver-1.2.0, platformdirs-4.13.0, cov-7.1.0, hypothesis-6.169.0, asyncio-1.4.0, benchmark-5.3.0, anyio-4.15.1
asyncio: mode=Mode.STRICT, debug=False, asyncio_default_fixture_loop_scope=None, asyncio_default_test_loop_scope=function
collecting ... collected 430 items

tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_empty_chunks_list_returns_empty XFAIL [  0%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_normal_chunks_list_processes PASSED [  0%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_batches_chunks_correctly PASSED [  0%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_returns_chunk_and_embedding_id_tuples PASSED [  0%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_embedding_provider_called_with_texts PASSED [  1%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_chunk_metadata_preserved PASSED [  1%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_batch_size_limit PASSED [  1%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_many_chunks PASSED [  1%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_embedding_provider_exception_propagates PASSED [  2%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_store_embedding_called_for_each_chunk PASSED [  2%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_chunk_text_extraction PASSED [  2%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_dismissive_bootcamp_language_detected XFAIL [  2%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_bootcamp_lacks_rigor_detected XFAIL [  3%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_bootcamp_inadequate_training_detected PASSED [  3%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_self_taught_comparison_detected PASSED [  3%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_positive_bootcamp_mention_not_flagged PASSED [  3%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_neutral_bootcamp_mention_not_flagged PASSED [  3%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_educational_background_positive_not_flagged PASSED [  4%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_demographic_assumption_age_detected XFAIL [  4%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_demographic_assumption_old_detected PASSED [  4%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_demographic_assumption_background_detected PASSED [  4%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_immigrant_developer_assumption_detected PASSED [  5%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_international_developer_assumption_detected PASSED [  5%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_clean_feedback_not_flagged PASSED [  5%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_technical_feedback_not_flagged PASSED [  5%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_development_suggestion_not_flagged PASSED [  6%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_case_insensitive_detection PASSED [  6%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_return_value_structure PASSED [  6%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_unbiased_returns_empty_reason PASSED [  6%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_biased_returns_reason PASSED [  6%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_empty_text PASSED [  7%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_whitespace_only PASSED [  7%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_coding_bootcamp_variant XFAIL [  7%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_online_course_bias_detected PASSED [  7%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_developer_vs_programmer_distinction XFAIL [  8%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_multiple_bias_indicators XFAIL [  8%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_negative_educational_claim XFAIL [  8%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_working_class_assumption PASSED [  8%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_rich_poor_assumption XFAIL [  9%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_foreign_developer_struggle_assumption PASSED [  9%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_skill_assessment_not_biased PASSED [  9%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_comparative_without_bias PASSED [  9%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_assumption_vs_observation XFAIL [ 10%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_feedback_fully_supported_by_context PASSED [ 10%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_feedback_with_no_support_in_context PASSED [ 10%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_partial_support_returns_middle_score XFAIL [ 10%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_empty_feedback_returns_zero PASSED [ 10%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_empty_context_chunks_returns_zero PASSED [ 11%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_both_empty_returns_zero PASSED [ 11%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_multiple_context_chunks XFAIL [ 11%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_extract_claims PASSED [ 11%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_extract_claims_with_punctuation PASSED [ 12%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_is_supported_with_keyword_overlap PASSED [ 12%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_is_supported_without_keywords PASSED [ 12%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_case_insensitive_support_check PASSED [ 12%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_score_never_returns_hardcoded_value PASSED [ 13%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_multiple_claims_varying_support XFAIL [ 13%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_very_long_feedback PASSED [ 13%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_very_long_context PASSED [ 13%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_common_words_filtered_in_overlap PASSED [ 13%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_minimum_overlap_required PASSED [ 14%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_none_context_chunk_text XFAIL [ 14%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_missing_text_key_in_chunk PASSED [ 14%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_score_consistency PASSED [ 14%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_specialized_technical_terms PASSED [ 15%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_results_sorted_by_score_descending PASSED [ 15%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_query_matching_no_documents PASSED [ 15%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_query_matching_multiple_documents PASSED [ 15%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_case_insensitive_matching PASSED [ 16%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_top_k_limit PASSED [ 16%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_top_k_larger_than_results PASSED [ 16%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_results_have_bm25_score PASSED [ 16%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_results_preserve_chunk_fields PASSED [ 16%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_empty_index XFAIL [ 17%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_index_not_called_returns_empty PASSED [ 17%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_multi_word_query PASSED [ 17%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_tokenization PASSED [ 17%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_tokenization_case_handling PASSED [ 18%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_large_corpus PASSED [ 18%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_special_characters_in_query PASSED [ 18%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_exact_phrase_matching PASSED [ 18%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_single_word_chunks PASSED [ 19%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_returns_list_of_floats PASSED [ 19%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_openai_provider_would_return_floats PASSED [ 19%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_deterministic PASSED [ 19%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_embedding_dimension PASSED [ 20%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_providers_same_dimension PASSED [ 20%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_multiple_texts PASSED [ 20%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_empty_list PASSED [ 20%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_single_text PASSED [ 20%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_empty_string PASSED [ 21%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_different_inputs_different_outputs PASSED [ 21%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_embeddings_normalized PASSED [ 21%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_provider_interface_implements_embed PASSED [ 21%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_returns_floats_not_integers PASSED [ 22%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_multiple_providers_same_interface PASSED [ 22%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_consistency_across_calls PASSED [ 22%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_provider_embed_signature PASSED [ 22%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_with_very_long_text PASSED [ 23%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_with_special_characters PASSED [ 23%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_with_unicode PASSED [ 23%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_with_newlines PASSED [ 23%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_with_tabs PASSED [ 23%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_different_text_lengths_same_output_dimension PASSED [ 24%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_batch_processing PASSED [ 24%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_embedding_vector_values_in_valid_range PASSED [ 24%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_providers_inherit_from_abstract_base PASSED [ 24%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_abstract_base_class_has_embed_method PASSED [ 25%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_embedding_stability PASSED [ 25%]
tests/unit/test_output_parser.py::TestOutputParser::test_json_wrapped_in_code_fence PASSED [ 25%]
tests/unit/test_output_parser.py::TestOutputParser::test_raw_json_without_fence PASSED [ 25%]
tests/unit/test_output_parser.py::TestOutputParser::test_plain_text_fallback PASSED [ 26%]
tests/unit/test_output_parser.py::TestOutputParser::test_malformed_json_fallback PASSED [ 26%]
tests/unit/test_output_parser.py::TestOutputParser::test_feedback_section_has_required_fields PASSED [ 26%]
tests/unit/test_output_parser.py::TestOutputParser::test_json_with_multiple_sections PASSED [ 26%]
tests/unit/test_output_parser.py::TestOutputParser::test_confidence_scores_in_sections PASSED [ 26%]
tests/unit/test_output_parser.py::TestOutputParser::test_empty_json_object PASSED [ 27%]
tests/unit/test_output_parser.py::TestOutputParser::test_json_array_fallback XFAIL [ 27%]
tests/unit/test_output_parser.py::TestOutputParser::test_very_long_plain_text PASSED [ 27%]
tests/unit/test_output_parser.py::TestOutputParser::test_html_in_feedback PASSED [ 27%]
tests/unit/test_output_parser.py::TestOutputParser::test_code_in_feedback PASSED [ 28%]
tests/unit/test_output_parser.py::TestOutputParser::test_section_suggestions_extraction PASSED [ 28%]
tests/unit/test_output_parser.py::TestOutputParser::test_unicode_characters PASSED [ 28%]
tests/unit/test_output_parser.py::TestOutputParser::test_nested_json_structure PASSED [ 28%]
tests/unit/test_output_parser.py::TestOutputParser::test_mixed_content PASSED [ 29%]
tests/unit/test_output_parser.py::TestOutputParser::test_plaintext_output_helper PASSED [ 29%]
tests/unit/test_output_parser.py::TestOutputParser::test_multiple_code_fences PASSED [ 29%]
tests/unit/test_output_parser.py::TestOutputParser::test_no_suggestions_key PASSED [ 29%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_email_redaction PASSED [ 30%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_multiple_emails_redacted PASSED [ 30%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_us_phone_number_redaction XFAIL [ 30%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_us_phone_formats XFAIL [ 30%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_international_phone_redaction PASSED [ 30%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_ssn_redaction PASSED [ 31%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_ssn_variations PASSED [ 31%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_street_address_redaction PASSED [ 31%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_text_with_no_pii PASSED [ 31%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_returns_list_of_pii PASSED [ 32%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_email_pii PASSED [ 32%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_phone_pii XFAIL [ 32%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_ssn_pii PASSED [ 32%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_positions_accurate PASSED [ 33%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_multiple_pii_items PASSED [ 33%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_case_insensitive_email_matching PASSED [ 33%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_complex_email_addresses PASSED [ 33%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_phone_at_start_of_text XFAIL [ 33%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_phone_at_end_of_text PASSED [ 34%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_address_variations PASSED [ 34%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_empty_text PASSED [ 34%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_whitespace_only PASSED [ 34%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_mixed_pii_and_text XFAIL [ 35%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_scrub_idempotent PASSED [ 35%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_no_false_positives PASSED [ 35%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_newline_injection_detected PASSED [ 35%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_system_role_switching_detected PASSED [ 36%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_human_role_switching_detected PASSED [ 36%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_assistant_role_switching_detected PASSED [ 36%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_template_injection_detected PASSED [ 36%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_jinja_injection_detected PASSED [ 36%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_ignore_instruction_detected PASSED [ 37%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_forget_instruction_detected PASSED [ 37%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_disregard_instruction_detected PASSED [ 37%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_override_instruction_detected PASSED [ 37%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_code_execution_attempt_detected PASSED [ 38%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_run_function_attempt_detected PASSED [ 38%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_eval_function_attempt_detected PASSED [ 38%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_clean_input_not_flagged PASSED [ 38%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_normal_text_not_flagged PASSED [ 39%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_sanitize_removes_template_delimiters PASSED [ 39%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_sanitize_removes_angle_brackets PASSED [ 39%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_sanitize_preserves_legitimate_content PASSED [ 39%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_sanitize_multiple_template_delimiters PASSED [ 40%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_separator_line_detected PASSED [ 40%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_long_separator_detected PASSED [ 40%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_case_insensitive_detection PASSED [ 40%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_whitespace_variations_detected XFAIL [ 40%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_multiple_injection_patterns PASSED [ 41%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_benign_mentions_not_flagged PASSED [ 41%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_empty_string PASSED [ 41%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_whitespace_only PASSED [ 41%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_sanitize_idempotent PASSED [ 42%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_complex_injection_attempt PASSED [ 42%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_legitimate_newlines_not_flagged PASSED [ 42%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_code_blocks_handled PASSED [ 42%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_sanitize_with_mixed_delimiters PASSED [ 43%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_all_5_templates_exist PASSED [ 43%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_skills_feedback_template_exists PASSED [ 43%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_projects_feedback_template_exists PASSED [ 43%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_presentation_feedback_template_exists PASSED [ 43%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_gaps_feedback_template_exists PASSED [ 44%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_first_impression_template_exists PASSED [ 44%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_each_template_contains_context_placeholder PASSED [ 44%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_each_template_contains_github_username_placeholder PASSED [ 44%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_each_template_contains_project_count_placeholder PASSED [ 45%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_returns_correct_template PASSED [ 45%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_projects_feedback PASSED [ 45%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_presentation_feedback PASSED [ 45%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_gaps_feedback PASSED [ 46%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_first_impression PASSED [ 46%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_unknown_name_raises_error PASSED [ 46%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_unknown_version_raises_error PASSED [ 46%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_all_templates_have_v1 PASSED [ 46%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_templates_are_strings PASSED [ 47%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_templates_have_reasonable_length PASSED [ 47%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_skills_feedback_mentions_technical_skills PASSED [ 47%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_projects_feedback_mentions_code_quality PASSED [ 47%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_gaps_feedback_mentions_missing_skills PASSED [ 48%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_presentation_feedback_mentions_readme PASSED [ 48%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_first_impression_is_concise PASSED [ 48%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_default_version PASSED [ 48%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_template_snapshot_content_hash PASSED [ 49%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_skills_feedback_requests_json_format PASSED [ 49%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_projects_feedback_requests_json_format PASSED [ 49%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_presentation_feedback_requests_json_format PASSED [ 49%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_gaps_feedback_requests_json_format PASSED [ 50%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_first_impression_plain_text PASSED [ 50%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_templates_have_portfolio_context PASSED [ 50%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_template_get_logs_retrieval PASSED [ 50%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_each_template_name_is_valid_identifier PASSED [ 50%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_template_versions_are_strings PASSED [ 51%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_no_hardcoded_usernames_in_templates PASSED [ 51%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_templates_use_consistent_placeholders PASSED [ 51%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_first_request_allowed PASSED [ 51%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_requests_up_to_limit_allowed PASSED [ 52%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_request_at_limit_plus_one_denied PASSED [ 52%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_remaining_count_correct PASSED [ 52%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_rolling_window_removes_old_entries PASSED [ 52%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_different_identifiers_independent PASSED [ 53%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_key_format_correct PASSED [ 53%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_entry_added_to_redis PASSED [ 53%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_expiry_set_correctly PASSED [ 53%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_custom_window_size PASSED [ 53%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_redis_error_handling PASSED [ 54%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_zero_limit PASSED [ 54%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_negative_limit PASSED [ 54%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_large_limit PASSED [ 54%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_return_tuple_structure PASSED [ 55%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_multiple_requests_same_user PASSED [ 55%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_time_based_window_calculation PASSED [ 55%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_ip_address_as_identifier PASSED [ 55%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_api_key_as_identifier PASSED [ 56%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_standard_readme XFAIL [ 56%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_longer_than_10000_chars PASSED [ 56%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_with_code_blocks PASSED [ 56%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_without_code_blocks PASSED [ 56%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_empty_readme PASSED [ 57%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_with_only_whitespace PASSED [ 57%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_bytes_input PASSED [ 57%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_bytes_with_utf8 PASSED [ 57%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_invalid_content_type PASSED [ 58%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_extract_heading_hierarchy XFAIL [ 58%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_extract_heading_hierarchy_no_headings PASSED [ 58%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_with_badges PASSED [ 58%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_without_badges PASSED [ 59%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_metadata_structure PASSED [ 59%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_word_count_accuracy PASSED [ 59%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_readme_with_all_quality_signals XFAIL [ 59%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_readme_with_no_content PASSED [ 60%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_readme_with_only_title PASSED [ 60%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_word_count_category_minimal PASSED [ 60%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_word_count_category_adequate PASSED [ 60%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_word_count_category_comprehensive PASSED [ 60%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_installation_section_detection PASSED [ 61%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_usage_section_detection PASSED [ 61%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_setup_keyword_counts_as_installation PASSED [ 61%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_quickstart_counts_as_usage PASSED [ 61%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_badge_detection PASSED [ 62%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_demo_link_detection PASSED [ 62%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_tech_stack_section_detection PASSED [ 62%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_technologies_keyword_counts PASSED [ 62%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_overall_score_calculation PASSED [ 63%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_missing_readme_content_key PASSED [ 63%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_result_has_all_required_fields PASSED [ 63%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_case_insensitive_section_detection PASSED [ 63%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_whitespace_only_readme PASSED [ 63%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_example_keyword_counts_as_usage PASSED [ 64%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_score_scales_with_word_count PASSED [ 64%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_try_it_as_demo_indicator PASSED [ 64%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_built_with_counts_as_tech_stack PASSED [ 64%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_query_with_perfect_keyword_match PASSED [ 65%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_query_with_zero_keyword_overlap PASSED [ 65%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_query_with_partial_overlap XFAIL [ 65%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_empty_chunks_list_returns_zero PASSED [ 65%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_empty_query_returns_zero PASSED [ 66%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_multiple_chunks_aggregated PASSED [ 66%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_case_insensitive_matching PASSED [ 66%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_tokenization PASSED [ 66%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_empty_text_in_chunk PASSED [ 66%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_very_long_query PASSED [ 67%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_very_long_chunk PASSED [ 67%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_special_characters_ignored PASSED [ 67%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_multiple_keyword_matches PASSED [ 67%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_single_word_chunks PASSED [ 68%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_score_ranges_from_zero_to_one PASSED [ 68%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_common_words_not_preventing_scoring PASSED [ 68%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_chunk_without_text_key PASSED [ 68%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_whitespace_only_in_chunks PASSED [ 69%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_average_relevance_calculation PASSED [ 69%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_single_column_resume_text PASSED [ 69%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_resume_no_work_experience PASSED [ 69%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_multipage_pdf PASSED [ 70%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_markdown_resume PASSED [ 70%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_invalid_content_type PASSED [ 70%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_invalid_list_content PASSED [ 70%]
tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections PASSED [ 70%]
tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections_matches_unindented_text[    ] PASSED [ 71%]
tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections_matches_unindented_text[\t] PASSED [ 71%]
tests/unit/test_resume_parser.py::TestResumeParser::test_strip_markdown_syntax PASSED [ 71%]
tests/unit/test_resume_parser.py::TestResumeParser::test_pdf_parsing_error_handling PASSED [ 71%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_preserves_text_content PASSED [ 72%]
tests/unit/test_review_service.py::TestReviewService::test_create_review_returns_review_with_pending_status PASSED [ 72%]
tests/unit/test_review_service.py::TestReviewService::test_get_review_returns_review_for_correct_owner XFAIL [ 72%]
tests/unit/test_review_service.py::TestReviewService::test_get_review_returns_none_for_wrong_user XFAIL [ 72%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_returns_paginated_results XFAIL [ 73%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_page_2_returns_correct_offset XFAIL [ 73%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_returns_tuple XFAIL [ 73%]
tests/unit/test_review_service.py::TestReviewService::test_create_review_calls_db_add PASSED [ 73%]
tests/unit/test_review_service.py::TestReviewService::test_create_review_calls_db_commit PASSED [ 73%]
tests/unit/test_review_service.py::TestReviewService::test_create_review_calls_db_refresh PASSED [ 74%]
tests/unit/test_review_service.py::TestReviewService::test_get_review_uses_select_and_join XFAIL [ 74%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_default_pagination XFAIL [ 74%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_custom_page_size XFAIL [ 74%]
tests/unit/test_review_service.py::TestReviewService::test_create_review_with_uuid_ids PASSED [ 75%]
tests/unit/test_review_service.py::TestReviewService::test_get_review_verifies_ownership XFAIL [ 75%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_counts_total XFAIL [ 75%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_returns_reviews_list XFAIL [ 75%]
tests/unit/test_review_service.py::TestReviewService::test_review_sections_and_score_initially_none PASSED [ 76%]
tests/unit/test_review_service.py::TestReviewService::test_get_review_with_valid_uuid XFAIL [ 76%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_ordered_by_created_at XFAIL [ 76%]
tests/unit/test_security.py::TestSecurity::test_hash_password_returns_bcrypt_hash PASSED [ 76%]
tests/unit/test_security.py::TestSecurity::test_verify_password_correct PASSED [ 76%]
tests/unit/test_security.py::TestSecurity::test_verify_password_incorrect PASSED [ 77%]
tests/unit/test_security.py::TestSecurity::test_verify_password_case_sensitive PASSED [ 77%]
tests/unit/test_security.py::TestSecurity::test_hash_same_password_different_hash PASSED [ 77%]
tests/unit/test_security.py::TestSecurity::test_create_access_token_returns_string PASSED [ 77%]
tests/unit/test_security.py::TestSecurity::test_create_access_token_is_jwt PASSED [ 78%]
tests/unit/test_security.py::TestSecurity::test_decode_access_token_valid PASSED [ 78%]
tests/unit/test_security.py::TestSecurity::test_decode_access_token_invalid_token PASSED [ 78%]
tests/unit/test_security.py::TestSecurity::test_decode_access_token_malformed PASSED [ 78%]
tests/unit/test_security.py::TestSecurity::test_decode_access_token_empty_string PASSED [ 79%]
tests/unit/test_security.py::TestSecurity::test_roundtrip_token_with_data PASSED [ 79%]
tests/unit/test_security.py::TestSecurity::test_create_access_token_with_custom_expiry PASSED [ 79%]
tests/unit/test_security.py::TestSecurity::test_access_token_includes_expiration PASSED [ 79%]
tests/unit/test_security.py::TestSecurity::test_password_hash_different_for_different_passwords PASSED [ 80%]
tests/unit/test_security.py::TestSecurity::test_verify_password_with_empty_strings PASSED [ 80%]
tests/unit/test_security.py::TestSecurity::test_verify_password_with_special_characters PASSED [ 80%]
tests/unit/test_security.py::TestSecurity::test_token_with_empty_data PASSED [ 80%]
tests/unit/test_security.py::TestSecurity::test_token_with_special_characters_in_data PASSED [ 80%]
tests/unit/test_security.py::TestSecurity::test_token_with_unicode_data PASSED [ 81%]
tests/unit/test_security.py::TestSecurity::test_hash_password_long_input PASSED [ 81%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format XFAIL [ 81%]
tests/unit/test_security.py::TestSecurity::test_token_tampering_detection PASSED [ 81%]
tests/unit/test_security.py::TestSecurity::test_create_token_consistency PASSED [ 82%]
tests/unit/test_security.py::TestSecurity::test_password_with_whitespace PASSED [ 82%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_empty_input_returns_empty_list PASSED [ 82%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_whitespace_only_input PASSED [ 82%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_short_text_returns_single_chunk PASSED [ 83%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_long_text_returns_multiple_chunks PASSED [ 83%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_chunk_metadata_includes_chunk_index PASSED [ 83%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_chunks_preserve_source_metadata PASSED [ 83%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_chunk_has_character_positions PASSED [ 83%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_bullet_points_not_split_mid_bullet PASSED [ 84%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_sentence_splitting PASSED [ 84%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_sentence_splitting_with_exclamation_marks PASSED [ 84%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_sentence_splitting_preserves_punctuation PASSED [ 84%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_overlap_between_chunks PASSED [ 85%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_chunks_have_non_zero_text PASSED [ 85%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_paragraph_boundaries_respected PASSED [ 85%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_metadata_copied_not_mutated PASSED [ 85%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_chunk_indexing_is_sequential PASSED [ 86%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_text_with_python_imports PASSED [ 86%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_text_with_python_type_annotations PASSED [ 86%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_text_with_typescript_files XFAIL [ 86%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_jupyter_ipynb_detection PASSED [ 86%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_mixed_language_text PASSED [ 87%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_frameworks_detected_with_high_confidence PASSED [ 87%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_react_detection PASSED [ 87%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_database_technology_detection XFAIL [ 87%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_devops_tool_detection XFAIL [ 88%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_skill_has_evidence_list PASSED [ 88%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_filename_based_detection PASSED [ 88%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_javascript_detection XFAIL [ 88%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_docker_compose_detection XFAIL [ 89%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_aws_gcp_azure_detection PASSED [ 89%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_empty_text PASSED [ 89%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_unrecognized_language PASSED [ 89%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_confidence_scores_are_floats PASSED [ 90%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_skill_detection_dataclass PASSED [ 90%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_empty_input_returns_empty_list PASSED [ 90%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_whitespace_only_input PASSED [ 90%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_document_with_no_headings XFAIL [ 90%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_document_with_nested_headings PASSED [ 91%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_heading_path_format PASSED [ 91%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_large_section_sub_chunked PASSED [ 91%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_chunk_metadata_includes_heading_level PASSED [ 91%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_chunk_metadata_structure PASSED [ 92%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_preserve_source_metadata PASSED [ 92%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_multiple_h1_headings PASSED [ 92%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_heading_path_breadcrumb PASSED [ 92%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_chunks_have_text_content PASSED [ 93%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_section_extraction_with_multiple_levels PASSED [ 93%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_heading_not_in_middle_of_content PASSED [ 93%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_empty_sections_handled PASSED [ 93%]
tests/unit/test_tech_detector.py::TestTechDetector::test_single_language_repo PASSED [ 93%]
tests/unit/test_tech_detector.py::TestTechDetector::test_mixed_language_repo_python_primary PASSED [ 94%]
tests/unit/test_tech_detector.py::TestTechDetector::test_ipynb_counted_as_python_not_json PASSED [ 94%]
tests/unit/test_tech_detector.py::TestTechDetector::test_node_modules_excluded XFAIL [ 94%]
tests/unit/test_tech_detector.py::TestTechDetector::test_vendor_files_excluded PASSED [ 94%]
tests/unit/test_tech_detector.py::TestTechDetector::test_build_directory_excluded XFAIL [ 95%]
tests/unit/test_tech_detector.py::TestTechDetector::test_config_file_detection PASSED [ 95%]
tests/unit/test_tech_detector.py::TestTechDetector::test_dockerfile_detection PASSED [ 95%]
tests/unit/test_tech_detector.py::TestTechDetector::test_github_actions_detection PASSED [ 95%]
tests/unit/test_tech_detector.py::TestTechDetector::test_makefile_detection PASSED [ 96%]
tests/unit/test_tech_detector.py::TestTechDetector::test_typescript_detection PASSED [ 96%]
tests/unit/test_tech_detector.py::TestTechDetector::test_go_detection PASSED [ 96%]
tests/unit/test_tech_detector.py::TestTechDetector::test_rust_detection PASSED [ 96%]
tests/unit/test_tech_detector.py::TestTechDetector::test_java_detection PASSED [ 96%]
tests/unit/test_tech_detector.py::TestTechDetector::test_multiple_python_files PASSED [ 97%]
tests/unit/test_tech_detector.py::TestTechDetector::test_empty_file_list PASSED [ 97%]
tests/unit/test_tech_detector.py::TestTechDetector::test_no_files_key PASSED [ 97%]
tests/unit/test_tech_detector.py::TestTechDetector::test_result_structure PASSED [ 97%]
tests/unit/test_tech_detector.py::TestTechDetector::test_framework_detection PASSED [ 98%]
tests/unit/test_tech_detector.py::TestTechDetector::test_unknown_extensions PASSED [ 98%]
tests/unit/test_tech_detector.py::TestTechDetector::test_case_insensitive_extension_matching PASSED [ 98%]
tests/unit/test_tech_detector.py::TestTechDetector::test_multiple_extensions_same_file PASSED [ 98%]
tests/unit/test_tech_detector.py::TestTechDetector::test_all_languages_sorted PASSED [ 99%]
tests/unit/test_tech_detector.py::TestTechDetector::test_frameworks_sorted PASSED [ 99%]
tests/unit/test_tech_detector.py::TestTechDetector::test_ruby_detection PASSED [ 99%]
tests/unit/test_tech_detector.py::TestTechDetector::test_csharp_detection PASSED [ 99%]
tests/unit/test_tech_detector.py::TestTechDetector::test_cpp_detection PASSED [100%]/Users/rupeshvarma/.codex/.chatgpt-projects/g-p-6ab54d8209dc819183d574683d512174/codepath-pathreview/.venv/lib/python3.11/site-packages/_pytest/unraisableexception.py:33: RuntimeWarning: coroutine 'AsyncMockMixin._execute_mock_call' was never awaited
  gc.collect()
RuntimeWarning: Enable tracemalloc to get the object allocation traceback


=============================== warnings summary ===============================
core/config.py:7
  /Users/rupeshvarma/.codex/.chatgpt-projects/g-p-6ab54d8209dc819183d574683d512174/codepath-pathreview/core/config.py:7: PydanticDeprecatedSince20: Support for class-based `config` is deprecated, use ConfigDict instead. Deprecated in Pydantic V2.0 to be removed in V3.0. See Pydantic V2 Migration Guide at https://errors.pydantic.dev/2.14/migration/
    class Settings(BaseSettings):

.venv/lib/python3.11/site-packages/passlib/utils/__init__.py:854
  /Users/rupeshvarma/.codex/.chatgpt-projects/g-p-6ab54d8209dc819183d574683d512174/codepath-pathreview/.venv/lib/python3.11/site-packages/passlib/utils/__init__.py:854: DeprecationWarning: 'crypt' is deprecated and slated for removal in Python 3.13
    from crypt import crypt as _crypt

tests/unit/test_review_service.py::TestReviewService::test_list_reviews_returns_tuple
  /Library/Frameworks/Python.framework/Versions/3.11/lib/python3.11/enum.py:711: RuntimeWarning: coroutine 'AsyncMockMixin._execute_mock_call' was never awaited
    return cls.__new__(cls, value)
  Enable tracemalloc to get traceback where the object was allocated.
  See https://docs.pytest.org/en/stable/how-to/capture-warnings.html#resource-warnings for more info.

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
================= 382 passed, 48 xfailed, 3 warnings in 16.41s =================
```

### Integration tests

pytest exited 5 (no tests collected); make exited 2. The Unit 4 instructions explicitly identify this empty-suite result as expected. This is not a passing test suite.

```text
$ make test-integration
.venv/bin/pytest tests/integration -v -m integration
============================= test session starts ==============================
platform darwin -- Python 3.11.4, pytest-9.1.1, pluggy-1.6.0 -- /Users/rupeshvarma/.codex/.chatgpt-projects/g-p-6ab54d8209dc819183d574683d512174/codepath-pathreview/.venv/bin/python
cachedir: .pytest_cache
hypothesis profile 'default'
benchmark: 5.3.0 (defaults: timer=time.perf_counter disable_gc=False min_rounds=5 min_time=0.000005 max_time=1.0 calibration_precision=10 warmup=False warmup_iterations=100000)
rootdir: /Users/rupeshvarma/.codex/.chatgpt-projects/g-p-6ab54d8209dc819183d574683d512174/codepath-pathreview
configfile: pyproject.toml
plugins: pytest_httpserver-1.2.0, platformdirs-4.13.0, cov-7.1.0, hypothesis-6.169.0, asyncio-1.4.0, benchmark-5.3.0, anyio-4.15.1
asyncio: mode=Mode.STRICT, debug=False, asyncio_default_fixture_loop_scope=None, asyncio_default_test_loop_scope=function
collecting ... collected 0 items

============================ no tests ran in 0.12s =============================
make: *** [test-integration] Error 5
```

### Lint

Exit 0.

```text
$ make lint
.venv/bin/ruff check .
All checks passed!
```

### Type checking

Exit 0; 76 source files checked.

```text
$ make typecheck
.venv/bin/mypy api/ core/ ingestion/ rag/ agent/ safety/
Success: no issues found in 76 source files
```

### Formatting

Exit 0; read-only format check, 110 files unchanged.

```text
$ .venv/bin/black --check .
All done! ✨ 🍰 ✨
110 files would be left unchanged.
```

### Frontend tests

Exit 0: 18 tests passed across 2 files, using Node 22.14.0 and npm 10.9.2. Initial dependency installation failed on the restricted home cache, then a network reset; installation succeeded with a workspace cache on retry. The command below is the successful test run.

```text
$ cd frontend && npm test -- --run

> pathreview-frontend@0.1.0 test
> vitest --run


 RUN  v1.6.1 /Users/rupeshvarma/.codex/.chatgpt-projects/g-p-6ab54d8209dc819183d574683d512174/codepath-pathreview/frontend

 ✓ src/components/__tests__/ReviewSection.test.tsx  (10 tests) 158ms
 ✓ src/components/__tests__/ProfileForm.test.tsx  (8 tests) 1107ms

 Test Files  2 passed (2)
      Tests  18 passed (18)
   Start at  23:16:11
   Duration  1.98s (transform 79ms, setup 379ms, collect 159ms, tests 1.26s, environment 757ms, prepare 128ms)

npm notice
npm notice New major version of npm available! 10.9.2 -> 12.2.0
npm notice Changelog: https://github.com/npm/cli/releases/tag/v12.2.0
npm notice To update run: npm install -g npm@12.2.0
npm notice
```
