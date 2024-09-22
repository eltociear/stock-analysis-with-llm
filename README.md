## Automated Stock Analysis with LLMs and AWS Bedrock Agents

This project aims to develop an automated system for comprehensive stock analysis using balance sheet data, technical indicators, and news, powered by large language models (LLMs) like Claude 3 and leveraging the AWS Bedrock infrastructure.

> **Note:** The estimated cost per run is approximately $50.

### Key Features

**Stock Analyst Module**
- Weekly stock analysis on S&P 500, Nasdaq 100 and EURO STOXX 50, ranks each stock in its respective industry based on:
   - Balance sheet information
   - Technical indicators
   - Relevant news
- LLMs ranks stocks within their respective industries ans stores reasoning
- BUY/SELL recommendations for each stock
- Stores results in a database for further analysis and trend tracking.

**Portfolio Manager Module**
- Updates weekly portfolio with new stocks (BUY) or sells stocks (SELL) in the portfolio based on the stock analyst's recommendations and general market sentiment.
- Allows user prompts to influence the selection and weighting of stocks in the portfolio.

### Architecture Overview

![Architecture](documentation/architecture.png)

The stock analysis application leverages various AWS services and external APIs. The key components and steps involved are as follows:

1. The process is triggered by an AWS EventBridge event.
2. The event initiates a task within the AWS Elastic Container Service (ECS), which fetches earnings reports from the Yahoo Finance API.
3. A prompt is sent to an Amazon Bedrock Agent, which performs web searches and summarizes key data points and relevant news for each stock.
4. The Amazon Bedrock Claude service summarizes the news and relevant data gathered.
5. Earnings reports, news, and industry benchmarks are sent to Amazon Bedrock Claude to rank stocks within their industries.
6. The LLM's recommendation, along with the summarized data, is saved in an Amazon DynamoDB database.
7. The portfolio management process is triggered by an AWS EventBridge event.
8. A prompt is sent to an Amazon Bedrock Agent to collect general market news.
9. The LLM acts as a portfolio manager with the information provided by the analyst and updates the portfolio accordingly.

### Getting Started

1. Deploy the infrastructure:
   ```
   cd infrastructure
   cdk deploy
   ```
2. Run the Python script `infrastructure/deploy_agents.py` to set up the Amazon Bedrock Agents, as this is currently not supported by AWS CDK. You need to add the IAM Role created by CDK to allow the agent to invoke bedrock model to the script. If the script fails because of the alias creation, do step 3 below and then create the agent alias manually in the console.
3. Configure the Action Groups and Agent settings in the Amazon Bedrock Agent console.
   - Click on `Agents`->`InternetSearchAgent`->`Edit in Agent Builder`->`Additional settings`-> Enable User Input
   - Click on `Add Action Group`
     - Enter Action group name `InternetSearch`
     - `Description`: this action group is use to google specific inputs 
     - Select the existing Lambda function created with CDK.
     - `Define inline schema` and copy the content from `src/schema/internet-search-schema.json`
     - Save end exit

   - Click edit agent. Go to advanced prompts settings. Toggle on the **Override pre-processing template defaults** radio button. Also make sure the **Activate pre-processing template** radio button is enabled.
   - Under *prompt template editor*, you will notice that you now have access to control the pre-built prompts. Scroll down to until you see "Category D". Replace this category section with the following:

      ```text
     -Category D: Questions that can be answered by internet search, or assisted by our function calling agent using ONLY the functions it has been provided or arguments from within <conversation_history> or relevant arguments it can gather using the askuser function.
      ```
   - Scroll down and select **Save & Exit**.

4. Update the AgentId and Alias in `src/helper/helper.py` with the adentID and aliasID from the Bedrock Console and run `cdk deploy` again.

### Results

For results, see `documentation/RESULTS.md`.

### Prompt Engineering

To improve the performance and behavior of the models, you can modify the prompts in the file `src/schema/prompts.yaml`.

### Financial Data Collected

The project collects a comprehensive set of financial performance metrics, valuation and market position data, governance and risk factors, industry comparisons, and company information for each stock.

### References

- [Bedrock Agents Webscraper](https://github.com/build-on-aws/bedrock-agents-webscraper)
- [BFI Working Paper](https://bfi.uchicago.edu/wp-content/uploads/2024/05/BFI_WP_2024-65.pdf)

### License

This project is licensed under the MIT License. Feel free to use and modify the code as needed.
