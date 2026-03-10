1. PostgreSQL schema draft:

        CREATE TABLE users (
            id UUID PRIMARY KEY,
            email TEXT UNIQUE NOT NULL,
            password_hash TEXT,
            display_name TEXT,
            timezone TEXT DEFAULT 'UTC',
            onboarding_completed BOOLEAN DEFAULT FALSE,
            account_status TEXT DEFAULT 'active',
            created_at TIMESTAMPTZ DEFAULT NOW(),
            updated_at TIMESTAMPTZ DEFAULT NOW()
        );

        CREATE TABLE user_preferences (
            user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
            preferred_journaling_mode TEXT DEFAULT 'both',
            preferred_tone TEXT DEFAULT 'balanced',
            checkin_reminder_time TIME,
            weekly_insight_day TEXT DEFAULT 'sunday',
            crisis_support_enabled BOOLEAN DEFAULT TRUE,
            trusted_contact_prompt_enabled BOOLEAN DEFAULT TRUE,
            data_sharing_level TEXT DEFAULT 'minimal',
            notification_preferences JSONB DEFAULT '{}'::jsonb,
            updated_at TIMESTAMPTZ DEFAULT NOW()
        );

        CREATE TABLE consent_logs (
            id UUID PRIMARY KEY,
            user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
            consent_type TEXT NOT NULL,
            status TEXT NOT NULL,
            consent_version TEXT NOT NULL,
            source TEXT,
            created_at TIMESTAMPTZ DEFAULT NOW()
        );

        CREATE TABLE journal_entries (
            id UUID PRIMARY KEY,
            user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
            entry_title TEXT,
            entry_text TEXT NOT NULL,
            source TEXT DEFAULT 'free_write',
            prompt_text TEXT,
            entry_date DATE DEFAULT CURRENT_DATE,
            is_deleted BOOLEAN DEFAULT FALSE,
            created_at TIMESTAMPTZ DEFAULT NOW(),
            updated_at TIMESTAMPTZ DEFAULT NOW()
        );

        CREATE TABLE voice_notes (
            id UUID PRIMARY KEY,
            user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
            audio_url TEXT NOT NULL,
            duration_seconds INT,
            transcript_text TEXT,
            transcript_status TEXT DEFAULT 'pending',
            source TEXT DEFAULT 'free_voice',
            prompt_text TEXT,
            language_code TEXT DEFAULT 'en',
            is_deleted BOOLEAN DEFAULT FALSE,
            created_at TIMESTAMPTZ DEFAULT NOW()
        );

        CREATE TABLE journal_analyses (
            id UUID PRIMARY KEY,
            user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
            source_type TEXT NOT NULL,
            source_id UUID NOT NULL,
            summary_text TEXT,
            emotion_labels JSONB DEFAULT '[]'::jsonb,
            emotion_intensity JSONB DEFAULT '{}'::jsonb,
            trigger_labels JSONB DEFAULT '[]'::jsonb,
            cognitive_distortions JSONB DEFAULT '[]'::jsonb,
            coping_behaviors JSONB DEFAULT '[]'::jsonb,
            risk_level TEXT DEFAULT 'tier_0',
            confidence_score NUMERIC(4,3),
            needs_followup BOOLEAN DEFAULT FALSE,
            suggested_next_action TEXT,
            model_version TEXT,
            created_at TIMESTAMPTZ DEFAULT NOW()
        );

        CREATE TABLE daily_checkins (
            id UUID PRIMARY KEY,
            user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
            mood_score INT CHECK (mood_score BETWEEN 1 AND 10),
            energy_score INT CHECK (energy_score BETWEEN 1 AND 10),
            stress_score INT CHECK (stress_score BETWEEN 1 AND 10),
            sleep_quality_score INT CHECK (sleep_quality_score BETWEEN 1 AND 10),
            focus_score INT CHECK (focus_score BETWEEN 1 AND 10),
            social_connection_score INT CHECK (social_connection_score BETWEEN 1 AND 10),
            free_text_note TEXT,
            checkin_date DATE DEFAULT CURRENT_DATE,
            created_at TIMESTAMPTZ DEFAULT NOW()
        );

        CREATE TABLE intervention_library (
            id UUID PRIMARY KEY,
            slug TEXT UNIQUE NOT NULL,
            title TEXT NOT NULL,
            category TEXT NOT NULL,
            description TEXT,
            instructions_json JSONB NOT NULL,
            estimated_duration_minutes INT,
            active BOOLEAN DEFAULT TRUE
        );

        CREATE TABLE intervention_sessions (
            id UUID PRIMARY KEY,
            user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
            intervention_id UUID NOT NULL REFERENCES intervention_library(id),
            trigger_source TEXT,
            recommended_by_agent BOOLEAN DEFAULT FALSE,
            started_at TIMESTAMPTZ DEFAULT NOW(),
            completed_at TIMESTAMPTZ,
            completion_status TEXT DEFAULT 'started',
            pre_distress_score INT CHECK (pre_distress_score BETWEEN 1 AND 10),
            post_distress_score INT CHECK (post_distress_score BETWEEN 1 AND 10),
            helpfulness_rating INT CHECK (helpfulness_rating BETWEEN 1 AND 5),
            feedback_text TEXT
        );

        CREATE TABLE weekly_insights (
            id UUID PRIMARY KEY,
            user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
            week_start_date DATE NOT NULL,
            week_end_date DATE NOT NULL,
            summary_text TEXT NOT NULL,
            top_patterns_json JSONB DEFAULT '[]'::jsonb,
            top_triggers_json JSONB DEFAULT '[]'::jsonb,
            effective_interventions_json JSONB DEFAULT '[]'::jsonb,
            recommended_focus_text TEXT,
            confidence_notes TEXT,
            opened_at TIMESTAMPTZ,
            created_at TIMESTAMPTZ DEFAULT NOW()
        );

        CREATE TABLE memory_items (
            id UUID PRIMARY KEY,
            user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
            memory_type TEXT NOT NULL,
            key TEXT NOT NULL,
            value_text TEXT,
            value_json JSONB DEFAULT '{}'::jsonb,
            strength_score NUMERIC(4,3) DEFAULT 0.500,
            first_seen_at TIMESTAMPTZ DEFAULT NOW(),
            last_seen_at TIMESTAMPTZ DEFAULT NOW(),
            status TEXT DEFAULT 'active',
            source_type TEXT,
            source_id UUID
        );

        CREATE TABLE safety_events (
            id UUID PRIMARY KEY,
            user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
            source_type TEXT NOT NULL,
            source_id UUID NOT NULL,
            trigger_text_excerpt TEXT,
            risk_level TEXT NOT NULL,
            risk_category TEXT NOT NULL,
            classifier_score NUMERIC(5,4),
            action_taken TEXT NOT NULL,
            model_version TEXT,
            review_status TEXT DEFAULT 'unreviewed',
            created_at TIMESTAMPTZ DEFAULT NOW()
        );

        CREATE TABLE trusted_contacts (
            id UUID PRIMARY KEY,
            user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
            name TEXT NOT NULL,
            relationship TEXT,
            phone_number TEXT,
            email TEXT,
            message_template TEXT,
            is_primary BOOLEAN DEFAULT FALSE,
            created_at TIMESTAMPTZ DEFAULT NOW()
        );

        CREATE TABLE behavioral_signals (
            id UUID PRIMARY KEY,
            user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
            signal_type TEXT NOT NULL,
            signal_value_numeric NUMERIC,
            signal_value_json JSONB DEFAULT '{}'::jsonb,
            signal_date DATE NOT NULL,
            source TEXT NOT NULL,
            created_at TIMESTAMPTZ DEFAULT NOW()
        );

        CREATE TABLE model_runs (
            id UUID PRIMARY KEY,
            user_id UUID REFERENCES users(id) ON DELETE SET NULL,
            task_type TEXT NOT NULL,
            source_type TEXT,
            source_id UUID,
            model_name TEXT NOT NULL,
            model_version TEXT NOT NULL,
            input_hash TEXT,
            output_hash TEXT,
            latency_ms INT,
            created_at TIMESTAMPTZ DEFAULT NOW()
        ); 


2. Indexes I'm gonna add:

        CREATE INDEX idx_journal_entries_user_created_at
        ON journal_entries(user_id, created_at DESC);

        CREATE INDEX idx_voice_notes_user_created_at
        ON voice_notes(user_id, created_at DESC);

        CREATE INDEX idx_daily_checkins_user_checkin_date
        ON daily_checkins(user_id, checkin_date DESC);

        CREATE INDEX idx_weekly_insights_user_week_start
        ON weekly_insights(user_id, week_start_date DESC);

        CREATE INDEX idx_memory_items_user_type
        ON memory_items(user_id, memory_type);

        CREATE INDEX idx_safety_events_user_created_at
        ON safety_events(user_id, created_at DESC);

        CREATE INDEX idx_behavioral_signals_user_signal_date
        ON behavioral_signals(user_id, signal_date DESC);

        CREATE INDEX idx_journal_analyses_user_created_at
        ON journal_analyses(user_id, created_at DESC);

3. JSON field examples
        a. emotion_labels:
                ["anxiety", "overwhelm", "sadness"]

        b. emotion_intensity
                {
                    "anxiety": 0.84
                    "sadness": 0.52
                }

        c. trigger_labels
                ["exams", "poor_sleep", "social_conflict"]

        d. top_patterns_json
                [
                    "Stress tends to rise after nights with less than 6 hours of sleep",
                    "Journaling is associated with slightly better next-day mood"
                ]

        e. instructions_json
        {
            "steps": [
                "Inhale for 4 seconds",
                "Hold for 4 seconds",
                "Exhale for 4 seconds",
                "Repeat 4 times"
            ],
            "type": "breathing"
        }
        



