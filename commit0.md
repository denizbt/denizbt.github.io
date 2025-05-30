---
layout: writeups
title: "commit0 (Autonomous Coding Agent)"
project_date: September 2024 - December 2024
project_role: Undergraduate Researcher
permalink: /research/commit0/
---

# Problem Statement

During this semester, I collaborated with a team of undergraduates to
develop an LLM agent capable of completing the
[commit0](https://github.com/commit-0/commit0) task. Our primary goal
was to create a system that could implement various Python packages
using only a skeletal codebase, access to a suite of unit test cases,
and some written documentation. The success of our LLM agents was
measured by the pass rate of the unit test suites for each Python
package. The project was conducted in a simulated GitHub environment
where the LLM agent made code edits, ran tests, and committed changes.

# Methods

Over the course of the semester, I explored several strategies for
developing the LLM agent. All work was based on the pair-programming
agent [aider](https://aider.chat/). Aider is an LLM agent which is
specifically fine-tuned to work in GitHub repositories, and can connect
to a number of different LLM models including OpenAI's GPT models and
Anthropic Claude.

The baseline approach presented in `commit0` was to implement each file
in a code repository individually, using a topological sort (of
dependencies) to get the implementation order. Baseline commit0 agents
used unit test case information to do their implementation, but did not
have iterative debugging features.

## Our Approach

Through extensive experimentation, I observed that the LLM agents rarely
produced correct code on the first attempt. This aligns with common
human programming practices, where debugging often consumes the majority
of development time. Inspired by these workflows, I focused on creating
a Debug Agent. This agent followed the same topological file
implementation order as the baseline `commit0` agent. It wrote code,
executed unit tests, and, most importantly, refined its implementations
based on test outcomes.

Here is a more detailed description of the approach I settled on at the
end of the semester.

1.  Implement functions in the next file $f$ according to the
    topological implementation order.

    - Prompt: \"Here is your task:

        You need to complete the implementations for all functions
        (i.e., those with pass statements) and pass the unit tests.

        Do not change the names of existing functions or classes, as
        they may be referenced from other code like unit tests, etc.

        When you generate code, you must maintain the original
        formatting of the function stubs (such as whitespaces),
        otherwise I will not able to search/replace blocks for code
        modifications, and therefore you will receive a score of 0 for
        your generated code.\"

2.  Run the unit test cases associated with file $f$ using `pytest` and
    collect the error output as a string $e$.

    -  The simple heuristic of checking whether the file name of $f$ is
        a substring of the test file name is used to identify the
        relevant test files to run.

3.  Prompt aider to modify the function code in file $f$ to fix the unit
    test case error. Aider automatically commits its changes after each
    prompt.

    -  Prompt: \"Modify or rewrite the functions implemented in the
        file $f$ to resolve the failed unit tests. The unit test output
        is: $e$\"

4.  Run `commit0 evaluate` for the entire repository. Compare the new
    unit test performance against the best performance from previous
    commits. If the new code passes the same or more unit tests than the
    best commit, update the best commit and proceed to the next file.
    Otherwise, revert to the previous best commit and continue with Step
    5.

5.  Repeat Steps 1-4 for file $f$ until the most recent commit improves
    performance or the system has attempted implementation for $f$ three
    times. If no improvement occurs after three attempts, retain the
    most recent commit and move to the next file.

    -  This step ensures that the system never move on from a file
        without saving a commit which includes an implementation for the
        current file. This prevents the system from not writing any new
        code, and terminating with the code base in its original,
        skeletal state.

## What Didn't Work

- **ManagerAgent**: Previous versions of the system included a
  ManagerAgent tasked with creating an implementation plan for the
  entire codebase. The ManagerAgent's plan was used instead of the
  default topological sort in `commit0`. However, the ManagerAgent's
  plans proved overly detailed, specifying the order not only for files,
  but also for individual functions within files. Furthermore, aider
  agent did not actually obey the plan; despite prompts to implement
  specific functions, the aider agent frequently made edits across the
  entire file.

  - Manager Prompt: \"You are a manager in charge of writing a plan to
    complete the implementations for all functions (i.e., those with
    pass statements) and pass the unit tests. Write a plan of attack to
    implement the entire repo, keeping in mind the most effective order
    in which tasks should be implemented. Please output the plan in the
    format of a list of numbered steps. Each step should specify a file
    to edit and a high-level description of the change to make. Note
    that we only need to edit the files that contain functions with pass
    statements, ie. those in the current context. Give me ONLY the plan,
    with no extraneous text.\"

- **Debugging $n$ times, $n > 1$**: I experimented with executing the
  debugging step (Step 3) multiple times. Given the nondeterministic
  nature of LLMs, I hypothesized that repeated debugging might improve
  the likelihood of producing code that passes the unit test suite.
  However, running the debugging step $n = 3$ times significantly
  increased runtime, making the process impractical. The final
  implementation limits debugging to a single attempt per file.

# Results

The following results are from the \"lite\" split, or list of
repositories, defined by the `commit0` task. All of the repositories
were implemented by an Aider agent which was connected to `gpt-4o-mini`.
I provide a copy of the `.agent.yaml` file used to run our agent through
the `commit0` command line interface.

    add_import_module_to_context: true
    agent_name: aider
    max_iteration: 3
    max_lint_info_length: 10000
    max_repo_info_length: 10000
    max_spec_info_length: 10000
    max_unit_tests_info_length: 10000
    model_name: gpt-4o-mini
    pre_commit_config_path: .pre-commit-config.yaml
    record_test_for_each_commit: false
    run_entire_dir_lint: false
    run_tests: false
    use_lint_info: false
    use_repo_info: false
    use_spec_info: false
    use_topo_sort_dependencies: true
    use_unit_tests_info: false
    use_user_prompt: false
    user_prompt: 'Here is your task:

      You need to complete the implementations for all functions (i.e., those with pass
      statements) and pass the unit tests.

      Do not change the names of existing functions or classes, as they may be referenced
      from other code like unit tests, etc.

      When you generate code, you must maintain the original formatting of the function
      stubs (such as whitespaces), otherwise we will not able to search/replace blocks
      for code modifications, and therefore you will receive a score of 0 for your generated
      code.'

The table below shows the unit test case suite results for each repository in the lite split of `commit0`. As a note, these results were obtained from running each repository individually, not through using the default
\"lite\" split provided through `commit0`. The Runtime column shows the time in seconds that the agent spent on implementing & debugging the repository.

### Table 1: Unit Test Results on `commit0` Lite Split
<div style="display: flex; justify-content: center;">
  <table style="border-collapse: collapse; width: auto;">
    <thead>
      <tr>
        <th style="padding: 20px; text-align: left; border: 1px solid #ccc;"><strong>Repository</strong></th>
        <th style="padding: 20px; text-align: right; border: 1px solid #ccc;"><strong>Runtime (s)</strong></th>
        <th style="padding: 20px; text-align: right; border: 1px solid #ccc;"><strong>Passed / Total Tests</strong></th>
      </tr>
    </thead>
    <tbody>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>cachetools</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">977.98</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">165 / 215</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>deprecated</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">265.47</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">80 / 171</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>cookiecutter</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">1922.42</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">59 / 367</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>wcwidth</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">77.24</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">2 / 38</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>parsel</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">302.54</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">8 / 206</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>pyjwt</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">309.25</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">1 / 259</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>babel</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">4186.59</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">0 / 5663</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>chardet</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">ERR</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">0 / 376</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>imapclient</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">735.47</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">0 / 267</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>jinja</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">14021.88</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">0 / 851</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>marshmallow</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">514.75</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">0 / 1229</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>minitorch</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">897.11</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">0 / 230</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>portalocker</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">199.74</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">0 / 36</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>simpy</code><span style="color: red;">*</span></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">1354.52</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">0 / 140</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>tinydb</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">9257.47</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">0 / 201</td></tr>
      <tr><td style="padding: 20px; border: 1px solid #ccc;"><code>voluptuous</code></td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">426.86</td><td style="padding: 20px; text-align: right; border: 1px solid #ccc;">0 / 149</td></tr>
    </tbody>
  </table>
</div>

## Brief Analysis

<span style="color: red;">*</span> It is worth noting that since the agent is by
nature non-deterministic, some repositories have been implemented better
on some runs than shown in this result (i.e. other implementations of
simpy have passed 15/140 test cases).

These results do not represent any improvement over previous LLM agents
who were given this task. Many repositories pass none of their unit test
cases when evaluated by `commit0`; manual inspection showed that code
was written and edited by the agent in all of the repositories,
including those with a 0% pass rate.

In many of the 0% pass rate repositories (i.e. minitorch, babel), there
were \"module import\" errors which were caused by the test files
expecting certain functions to exist, or have a specific name; aider
either did not implement a function with that name or potentially
renamed functions. These fundamental import errors prevented many unit
test cases from every successfully executing.

Another issue is that if our agent wrote code with compile time errors,
then `pytest` cannot run the code at all. Repositories such as tinydb
contained compile-time errors after being implemented and thus were not
able to be evaluated; I manually ran tinydb through my system twice and
got 0% pass rate both times.

Another note is that this system does not do any pre processing of error
output before giving it to be used by the model; I assumed that the more
context which the model is given, the better it would perform. I suspect
that more targeted and purposeful debugging is crucial in order to
improve performance.

The primary difficulty I encountered when creating this system was in
ensuring that it was robust enough to work with many different
repositories. For instance, I could not rely on unit test cases having
descriptive names which could be directly matched to the function they
were testing. In fact, many unit test cases test multiple functions,
i.e. when one function calls another one. This makes it very difficult
to programmatically identify which file needs to be debugged.

# Running the Code

The results presented in the previous section were obtained from running
the code in the rollback-integration branch of the following [GitHub
repository](https://github.com/MaxWhitton25/multi-agent-repo-completion/tree/rollback-integration).
In order to run my code, you need to create a Python environment which
runs Python 3.12. This code was integrated with `commit0`, and is only
guaranteed to work with version 0.1.6. You can install `commit0` using
pip.

    % pip install commit0==0.1.6 

Now, assuming that your current directory is the root directory of
`multi-agent-repo-completion`, you should run
`run_system.py –split="lite"` script, replacing \"lite\" with the split
of repositories that the agent should implement. The current
implementation run with the \"lite\" split does sometimes (i.e. not
deterministically) incur an error within Aider,
`RecursionError: maximum recursion depth exceeded`. The exact source of
that error has not been ascertained, but it does not occur when
repositories are implemented individually. The results reported in
Section 3 were ascertained from running each repository individually to
confirm that this error did not affect them. For repositories with a 0%
unit test pass rate, their performance did not change at all when run
individually.

Upon the completion of the program, you can run\
`commit0 evaluate –branch="commit0"` to get a summary of the unit test
cases results for the split specified in the `.commit0.yaml` file in
your repository.