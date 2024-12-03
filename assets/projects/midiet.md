# Midiet: A Personalized Nutrition Program

### Overview of the Application

The project, titled **"miDiet: A Personalized Nutrition Program,"** was developed collaboratively by **me** and **Papadiochos Nikos** as part of our Master of Science (MSc) coursework in *"Software Personalization Technologies"*.

This app provides users with a personalized seven-day meal schedule based on their specific dietary preferences, nutritional needs, and health goals. It leverages recommender systems to analyze user input, such as dietary preferences, allergies, and caloric requirements, to propose balanced and diverse meal plans.

Key Features of the Application:
1. **Customization:** The app categorizes users based on their dietary preferences, dislikes, and allergens using a scoring system.
2. **Personalized Meal Plans:** Users receive meal recommendations tailored to their Body Mass Index (BMI), Total Daily Energy Expenditure (TDEE), and personal preferences.
3. **Alternative Suggestions:** If users encounter foods they dislike or cannot consume, the app offers substitutes with equivalent nutritional value.
4. **Feedback Integration:** Daily user feedback refines meal suggestions through a dynamic scoring system.
5. **Reward System:** After adhering to a weekly plan, users are rewarded with indulgent meal options, adding motivation to maintain their diet.
6. **User-Friendly Interface:** Options for dark and light themes, language settings, and a personalized dashboard enhance user interaction.

---

### Technical Details

#### 1. **Development Tools & Technologies**
- **Frontend:** Built with **Windows Presentation Foundation (WPF)** for a responsive and interactive user interface.
- **Backend:** Developed using the **.NET Framework with C#**, ensuring robustness and efficient processing.
- **Database:** Employs **Microsoft SQL Server** for secure and structured data storage and retrieval.

#### 2. **Architecture**
The app utilizes an **N-tier architecture**, where each layer operates independently:
- **Presentation Layer:** Manages user interface and experience.
- **Business Logic Layer:** Processes inputs and applies algorithms for meal recommendations.
- **Data Layer:** Handles database interactions.

#### 3. **Core Functionalities**
- **BMI and TDEE Calculations:** Determines user health metrics to categorize them into weight management goals (e.g., weight loss, maintenance, or gain).
- **Recommendation Algorithm:** Combines user data with a scoring system to rank food items based on preferences and restrictions.
- **Food Substitution:** Locates alternatives matching caloric and nutritional profiles using database queries.
- **Progress Monitoring:** Tracks adherence to plans through a **Result Calendar** and adjusts future recommendations accordingly.

#### 4. **Recommender Systems**
The system employs machine learning techniques, such as:
- **Stereotype Modeling:** Groups users with similar characteristics to simplify initial recommendations.
- **Dynamic Scoring:** Updates food item scores based on user feedback and behavior.

#### 5. **Interface Design**
- **Onboarding Wizard:** Captures initial user data for effective personalization.
- **Dashboard:** Summarizes user data and displays key metrics.
- **Diet Calendar:** Visualizes meal plans and allows tracking.
- **Customization Options:** Provides flexibility in theme, language, and layout preferences.
