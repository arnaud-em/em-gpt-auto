## Flow

* In `main.py`, enters the program
* In `autogpt/__main__.py`, calls the `main()` function
* In `autogpt/__main__.py`, in `main()`, it calls `construct_prompt()`
* In `autogpt.prompt.py`, in `construct_prompt` creates the `prompt` which is built in steps
  * First, in `autogpt.prompt.py`, in `construct_prompt()` the `AIConfig` is built. It contains 
    * Name
    * Role
    * Goals
  * Then, in `autogpt.ai_config.py`, `construct_full_prompt()` the first part of the prompt is built in with 
    * AI name
    * AI role
    * AI general instructions
    * AI goals
  * Finally, in `autogpt.prompt.py`, in `get_prompt()`, the rest of the prompt is built
    * constraints
    * commands
    * resources
    * performance evaluation
  * NB : This flow is confusing and should/will be refactored
* In `autogpt/__main__.py`, in `main()`, it calls `Agent().start_interaction_loop()`
* In `autogpt.agent.agent.py`, the `Agent` contains
  * Agent Name
  * Permanent Memory Store
  * Message History
  * Prompt
  * User Input
* In `autogpt.agent.agent.py`, in `Agent`, calls  `start_interaction_loop()` which does
  * Calls `chat_with_ai()` in `autogpt.chat.py` and collects the answer in `assistant_reply`
    * `prompt` built earlier
    * `user_input`: Initializes to a constant and then is what the user passes
    * `full_message_history`: Initializes to [] and the conatins the set of sequences
    * `permanent_memory`: Redis, LocalCache, Melvis, PineCone
    * `token_limit`: Comes form the config and is set to 4000 for GPT-3.5 and 8000 for GPT-4
  * Calls `attempt_to_fix_json_by_finding_outermost_brackets()` to try to fix the JSON as sometimes the AI does not return a proper JSON
  * Calls `get_command()` in `autogpt.app.py` to parse the AI response and get the command to execute
  * Then it asks for feedback from the user (if not ran in continuous mode)
    * Enter Y to tell the AI to proceed
    * Enter Y -N where N is an integer to run N continous commands
    * Enter N to exist the program
    * Enter plain text to provide feedback to the UI
  * Calls `execute_command()` in `autogpt.app.py` to execute the command
    * 
  * Then adds the following to the memory store as one blob of text
    * Assistant Reply
    * Results
    * Human Feedback
  * Also adds it to the message history
* In `autogpt.chat.py`, in `chat_with_ai()`, we see the following logic
  * Extracts the relevant memory from the message history with `get_relevant()`
    * Message history as a string
    * Relevance parameter (5 by default by always overwritten to 10)
    * Uses embeddings to extract the top K most relevant pieces of text
  * Uses `generate_context()` to create the full context
    * The prompt
    * The time
    * The memory retrieved via embeddings
    * The budget for the context is 2500 tokens, it removes memories till it goes below 2500 tokens
  * Then it adds the user input (which also consumes tokens)
  * Then it keeps adding the most recent message history to the context till it hits the token budget
    * The token budget is 4000 - 1000 (reserved for response) - 2500 or less (reserved for context) - X (user input) = 500
  * In `autogpt.llm_utils.py`, in `create_chat_completion`, we pass the model, the context and the remaining token limit
    * Calls the Chat GPT API to get the answer
    * The temperature is set via the `TEMPERATURE` property in the config