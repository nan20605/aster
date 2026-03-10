note: i've designed the data model such that it's clean enough to build now (for v1), but flexible enough for later additions like therapist dashboards, wearables, and rich safety analytics. 

goal:
Design the core entities Aster needs to support:
    onboarding
    journaling
    voice journaling
    daily check-ins
    interventions
    weekly insights
    safety escalation
    personalization and memory
    future health integrations

design principles:
a. separate raw data from interpreted data
    Example:
    raw journal text is stored as entered
    extracted emotions/triggers are stored separately
    this makes reprocessing possible when models improve
b. store events, not just state
    You want historical data for:
        trend analysis
        weekly insights
        personalization
        safety review
        product analytics
    So instead of only storing “current mood,” store each check-in as a timestamped event. 
c. keep safety auditable 
    Every risk-related event should be logged with:
    trigger source
    risk level
    action taken
    model version
d. make memory structured
    Do not store personalization as vague LLM notes only.
    Store reusable structured memory:
        common triggers
        recurring emotions
        preferred interventions
        helpful support style
e. Consent must be first-class
    For a product like this, permissions and consent history matter.

main entities:
    for v1, the entities are:
        users
        user_preferences
        consent_logs
        journal_entries
        voice_notes
        journal_analyses
        daily_checkins
        intervention_library
        intervention_sessions
        weekly_insights
        memory_items
        safety_events
        trusted_contacts
        behavioral_signals
        model_runs

entity-by-entity definition:
        1. users
            Stores the base user record.
            Key fields:
                id
                email
                password_hash or auth provider id
                display_name
                created_at
                updated_at
                timezone
                date_of_birth_year or age band if needed later
                onboarding_completed
                account_status
            Why:
                You need a central identity object for everything else.
        2. user_preferences
            Stores user-configurable app preferences.
            Key fields:
                user_id
                preferred_journaling_mode (text, voice, both)
                preferred_tone (gentle, direct, balanced)
                checkin_reminder_time
                weekly_insight_day
                crisis_support_enabled
                trusted_contact_prompt_enabled
                data_sharing_level
                notification_preferences
            Why:
                Preferences should not clutter the main users table.
        3. consent_logs
        Tracks consent and permissions over time.
            Key fields:
                id
                user_id
                consent_type
                terms_of_use
                privacy_policy
                microphone
                notifications
                health_data
                trusted_contact
                safety_disclaimer
                status (granted, revoked)
                consent_version
                timestamp
                source
            Why: 
                Critical for privacy, compliance, debugging, and trust.
        4. journal_entries
            Stores user-written text journals.
            Key fields
                id
                user_id
                entry_text
                entry_title (optional)
                source (free_write, prompted, follow_up)
                prompt_text (optional)
                created_at
                updated_at
                is_deleted
                entry_date
            Why
                Core raw journaling table.
        5. voice_notes
        Stores uploaded/recorded voice entries.
        Key fields
            id
            user_id
            audio_url
            duration_seconds
            transcript_text
            transcript_status
            source (free_voice, prompted_voice)
            prompt_text optional
            created_at
            language_code
            is_deleted
        Why
            Raw audio and transcript lifecycle should be distinct from journal analysis.
        6. journal_analyses
        Stores structured outputs from text or voice entry analysis.
        Key fields
            id
            user_id
            source_type (text_journal, voice_note)
            source_id
            summary_text
            emotion_labels JSON
            emotion_intensity JSON
            trigger_labels JSON
            cognitive_distortions JSON
            coping_behaviors JSON
            risk_level
            confidence_score
            needs_followup
            suggested_next_action
            model_version
            created_at
        Why
            Keeps interpreted outputs separate from raw user input.
        7. daily_checkins
        Stores lightweight daily self-report data.
        Key fields
            id
            user_id
            mood_score
            energy_score
            stress_score
            sleep_quality_score
            focus_score
            social_connection_score
            free_text_note (optional)
            created_at
            checkin_date
        Why
            This becomes the backbone for trend detection.
        8. intervention_library
        Stores all interventions available in the app.
        Key fields
            id
            slug
            title
            category
            breathing
            grounding
            cbt
            dbt
            sleep
            behavioral_activation
            description
            instructions_json
            estimated_duration_minutes
            active
        Why
            Interventions should be centrally stored and versionable.
        9. intervention_sessions
        Stores each time a user starts/completes an intervention.
        Key fields
            id
            user_id
            intervention_id
            trigger_source
            checkin
            journal
            voice
            manual
            safety_flow
            recommended_by_agent
            started_at
            completed_at
            completion_status
            pre_distress_score (optional)
            post_distress_score (optional)
            helpfulness_rating
            feedback_text (optional)
        Why
            This lets you learn what helps whom.
        10. weekly_insights
        Stores generated weekly summaries.
        Key fields
            id
            user_id
            week_start_date
            week_end_date
            summary_text
            top_patterns_json
            top_triggers_json
            effective_interventions_json
            recommended_focus_text
            confidence_notes
            opened_at optional
            created_at
        Why
            Keeps historical insights accessible and measurable.
        11. memory_items
        Stores longitudinal personalized memory.
        Key fields
            id
            user_id
            memory_type
                trigger
                emotion_pattern
                support_preference
                helpful_intervention
                recurring_theme
                risk_pattern
            key
            value_text
            value_json
            strength_score
            first_seen_at
            last_seen_at
            status (active, decayed, archived)
            source_type
            source_id
        Why
            This is the personalization engine.
        12. safety_events
        Stores all safety-related detections and routing decisions.
        Key fields
            id
            user_id
            source_type
            source_id
            trigger_text_excerpt
            risk_level
            risk_category
                self_harm
                suicidality
                panic
                hopelessness
                dependency
                abusive
                diagnosis_request
                medication_request
            classifier_score
            action_taken
                normal_flow
                supportive_response
                grounding_prompt
                trusted_contact_prompt
                crisis_resources_shown
                emergency_guidance_shown
            model_version
            review_status
            created_at
        Why
            This is essential for audits and improving safety.
        13. trusted_contacts
        Stores optional trusted contact setup.
        Key fields
            id
            user_id
            name
            relationship
            phone_number (optional)
            email (optional)
            message_template (optional)
            is_primary
            created_at
        Why
            Important for escalation workflows.
        14. behavioral_signals
        Stores non-journal behavioral data and derived signals. 
        Key fields
            id
            user_id
            signal_type
                sleep_duration
                sleep_consistency
                bedtime_drift
                activity_count
                journaling_frequency
                checkin_adherence
                screen_rhythm
                heart_rate
                hrv
            signal_value_numeric
            signal_value_json
            signal_date
            source
                self_report
                healthkit
                health_connect
                derived
            created_at
        Why
            Needed for time-series modeling and multimodal insights.
        15. model_runs
        Tracks model execution metadata.
        Key fields
            id
            user_id
            task_type
                summarization
                emotion_extraction
                risk_classification
                weekly_insight
                intervention_ranking
                memory_extraction
            source_type
            source_id
            model_name
            model_version
            input_hash
            output_hash
            latency_ms
            created_at
        Why
            Useful for debugging, reproducibility, and evals.

Relationships:
    The relational logic:
        one user has one user_preferences
        one user has many consent_logs
        one user has many journal_entries
        one user has many voice_notes
        one journal_entry can have many journal_analyses over time, but usually one active analysis
        one voice_note can have many journal_analyses over time
        one user has many daily_checkins
        one user has many intervention_sessions
        one intervention_library item can be used in many intervention_sessions
        one user has many weekly_insights
        one user has many memory_items
        one user has many safety_events
        one user has many trusted_contacts
        one user has many behavioral_signals
        one user has many model_runs

ER-style text diagram:
    users
    ├── user_preferences (1:1)
    ├── consent_logs (1:N)
    ├── journal_entries (1:N)
    │     └── journal_analyses (1:N via source_id/source_type)
    ├── voice_notes (1:N)
    │     └── journal_analyses (1:N via source_id/source_type)
    ├── daily_checkins (1:N)
    ├── intervention_sessions (1:N)
    │     └── intervention_library (N:1)
    ├── weekly_insights (1:N)
    ├── memory_items (1:N)
    ├── safety_events (1:N)
    ├── trusted_contacts (1:N)
    ├── behavioral_signals (1:N)
    └── model_runs (1:N)

To add later (but not in v1 --> thus not included in schema_draft):
    therapist accounts
    clinician notes
    appointment objects
    insurance metadata
    team/org tables
    advanced wearable session tables
    RL feedback tables
    full feature store
            