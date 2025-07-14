# SmartCall Intelligent Voice Assistant

**Project Overview:**  
SmartCall is a cloud-based intelligent voice assistant, integrating speech-to-text (STT), large language models (LLM), text-to-speech (TTS), and PostgreSQL for memory management and personalized user experience. Built with n8n for automated business workflow orchestration.

## Key Features

- End-to-end automation of AI-powered voice interactions
- Seamless integration of STT, LLM, and TTS APIs
- Real-time conversation memory with PostgreSQL
- Automated appointment scheduling and customer support workflow

## Technologies Used

- n8n (workflow automation)
- Azure Speech-to-Text, OpenRouter LLM, Azure Text-to-Speech
- PostgreSQL (memory database)
- Docker (deployment)

## My Contribution

- Led the overall architecture design and implementation
- Integrated and compared multiple AI models and APIs to optimize performance
- Built and maintained automated workflow using n8n
- Troubleshot and fixed system bugs, ensured production stability
- Documented system configuration and provided handover support

## Demo / Source Code


<img width="1064" height="416" alt="image" src="https://github.com/user-attachments/assets/7e02fb6c-398b-42d8-a40b-131648f27db5" />

We added some special features to make it smarter. For example:
- It can tell if the user wants to book a meeting. 
- It remembers past conversations with the same user. 
- It uses PostgreSQL to save data. You can update the prompt anytime. 
- You can also switch between LLM models.

## Prompt Engineering for Intent Recognition

The following prompt template and logic are used in the SmartCall workflow to detect user intention and ensure clear, human-like conversation flow. This approach guarantees that only explicit meeting booking requests are recognized by the assistant.

```javascript
// prompt.js
const inputMessages = $json.messages || [];
const userInput = inputMessages[inputMessages.length - 1]?.content || "";

const companyProfile = {
  name: "COMPANY",
  location: "LOCATION",
  services: [
    "PART_1",
    "PART_2",
    "PART_3"
  ],
  tone: "professional, polite, and conversational"
};

const systemPrompt = `
You are the official voice assistant of ${companyProfile.name}, a company based in ${companyProfile.location}.
Your tone is ${companyProfile.tone}.
You must respond naturally in spoken English, as if you are a human assistant.
The company offers services such as: ${companyProfile.services.join(", ")}.
Answer clearly, keep responses brief and relevant. Never say you're an AI.
Important: Do not use bullet points, numbered lists, or structured formatting.
Always write in full conversational sentences as if you are speaking directly to a customer.
Use contractions (e.g., "I'm", "we're", "you'll") to make your speech sound more natural.

Your only task is to identify the user’s intention — whether they clearly want to schedule a **new** meeting or not.

Rules:
- If the user **clearly says** they want to book, arrange, or schedule a new meeting, then respond politely and append: [Intention: book_meeting]
- If the user is unsure, asks a question, talks about an existing meeting, or you are the one offering a meeting, then respond politely and append: [Intention: other]

Important:
- You are NOT allowed to say the meeting is scheduled.
- You are NOT allowed to add anything to a calendar.
- You are NOT allowed to assume booking intent unless the user explicitly expresses it.
- You MUST rely only on the user's message — not your assumption.

Examples:
- User: "Can I book a meeting?" → [Intention: book_meeting]
- User: "I'm confused." or "What does this mean?" → [Intention: other]
- User: "That meeting was helpful." → [Intention: other]
- User: "Can you help me arrange a time?" → [Intention: book_meeting]

Only append one of these two tags at the end of your reply:
→ [Intention: book_meeting]  
→ [Intention: other]
`;

const messages = [
  { role: "system", content: systemPrompt },
  ...inputMessages
];

return [{ json: { ...$json, messages } }];

```

## Intent Detection and Meeting Time Extraction

The following function is used in our workflow to automatically determine whether the user intends to book a meeting, based on the AI assistant's response. It also extracts any meeting time information mentioned by the user.

```javascript
// intentParser.js
const rawContent =
  $json.content ||
  $json.text ||
  $json.choices?.[0]?.message?.content ||
  "";

const cleaned = rawContent.replace(/\n/g, " ");
const intentMatch = cleaned.match(/\[Intention:\s*(book_meeting|other)\]/i);
const intent = intentMatch ? intentMatch[1].toLowerCase() : "other";
const isBooking = intent === "book_meeting";

const timeMatch = cleaned.match(/\w+ \d{1,2}(?:th|st|nd|rd)? at \d{1,2} ?(AM|PM)/i);
const extracted_time = timeMatch ? new Date(timeMatch[0] + ", 2025").toISOString() : null;

return [{
  json: {
    ...$json,
    reply: rawContent,
    isBooking,
    extracted_time,
    debug_intent: intent
  }
}];

```

## Contact

For further details or demo requests, please contact:  
[Your Email]

