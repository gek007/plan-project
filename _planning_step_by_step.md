## Creating a plan Step by step 

1. **Create brainstorm.md file with goal of your app - high level design**  
   
2. **Ask "Claude Code"** 
   
    I want to build the project outlined in @project_instraction.md. 
    What are the four most important questions I need to answer in order to build this successfully? It created @questions.md file with all questions


   **Claude Answer** -> in @questions_by_claude.md  

   
3. **Ask GhatGPT:** 

    Hey, ChatGPT. I want to build a new app that is going to extract summary, decisions, action items, risks, and auto-creates tasks from uploaded meeting recording file.   

    I’m trying to think through the app functionality. What are some questions that I should think about in order to build this app and make it useful.  


   **GPT Answer :**   ->  in @questions_by_chatgpt.md  

    Make further conversation with ChatGPT about all questions 

4. **Project_spec.md doc (PSD)**  

      Key Components:

      Part 1: Product requirements

          - Who is the product for ?
          - What problems does it solve ?
          - What does the product do ?  

      Part 2: Technical design

          - Tech stack 
  
               * Language  
               * Frontend 
               * Backend
               * DB 
               * Cloud 
               * AI model     
               * Auth/login 
               * Emails 
               * Object storage   
               * 
          
          - Architecture 
          - System design   
          

5. **you can ask claude:**

        "Which image model would be best for my app. Create a research report comparing the the best image models available via API on quality, cost, and ease of use. Then recommend a choice to me. Output your findings in a doc called "research_report_image_model.md".


6. **What are milestones of functionality ?**

      - MVP 
      - v1 
      - v2 
      - Later    


7. **Setup checklist** 

      - Github repo 
      - .env file 
      - CLAUDE.md 
      - Plugins: slash commands, sub-agents, rules, skills, MCP servers, hooks   
    -         

    

    
