# ai-analytics

Аналитический модуль платформы «СкиллКомпас». Изолированный репозиторий для разработки приложения `apps/analytics`.

---

## Что это за репозиторий

Это **песочница** для аналитики. Здесь есть только те модели и настройки, которые нужны для работы над `apps/analytics`:

#### Из organizations
Organization — организация (ВУЗ, школа, ООО). Поля: inn, short_name, full_name, organization_type, district.

OrganizationType — тип организации («Общее образование», «ВУЗ», «IT» и т.д.).

District — район (муниципалитет).

#### Из accounts
EmployeeProfile — профиль работника. Поля: organization, position, position_level, education_level, work_experience_years, is_unemployed.

Position — справочник должностей (привязка к типу организации).

#### Из surveys
SurveySession — прохождение опроса. Поля: user, category, status, completed_at.

DiagnosticKitResult — результаты диагностического комплекта (4 работы + итог). Поля: work1_score…work4_score, total_score.

UserSurveyResponse — ответы на вопросы. Поля: question, score, group, selected_option.

SurveyQuestionOption — варианты ответа с is_correct и group (А/Б/В/Г).

#### Из courses
Course — курс. Поля: title, organization, url, is_deleted, duration_hours.

Category — категория курса.

#### Из recommendations
CourseRecommendation — рекомендация курса работнику. Поля: user, course, relevance_score.

SelectedCourse — выбранный работником курс. Поля: user, course.

ApprovedSelectedCourse — одобрение курса работодателем. Поля: selected_course, status, approved_by.


---

## Технический стек

- Python 3.14.5
- Django 6.0.5
- PostgreSQL 18
- nginx 1.29.8
- Redis 8 (для Celery)
- Django REST Framework
- pytest, pytest-django
- Docker, Docker Compose

---

## Структура проекта
```
ai-analytics/
├── ai/                       # настройки Django
│   ├── settings.py           
│   ├── urls.py
├── apps/
│   ├── users/                
│   ├── organizations/        
│   ├── accounts/             
│   ├── surveys/              
│   ├── courses/              
│   ├── recommendations/      
│   └── analytics/            # ТВОЯ ЗОНА
│       ├── models.py
│       ├── services/
│       ├── views.py
│       ├── urls.py
│       ├── admin.py
│       ├── tasks.py
│       ├── migrations/
│       └── tests/
├── docker/
├── docker-compose.yml
├── requirements.txt
├── .env.example
├── pytest.ini
└── README.md
```

## Примеры запросов
#### Найти всех работников школы с должностью «Директор»
```
from apps.accounts.models import EmployeeProfile

directors = EmployeeProfile.objects.filter(
    organization__organization_type__name="Общее образование",
    position="Директор школы",
)
```

#### Найти результаты диагностики по организации
```
from apps.surveys.models import DiagnosticKitResult

results = DiagnosticKitResult.objects.filter(
    session__user__organization_id=organization_id,
    session__status="completed",
)
```

#### Посчитать, сколько раз курс рекомендовался
```
from apps.recommendations.models import CourseRecommendation

count = CourseRecommendation.objects.filter(course_id=course_id).count()
```

#### Найти курсы, которые выбирали, но не одобряли
```
from apps.recommendations.models import SelectedCourse

pending = SelectedCourse.objects.filter(
    approval__status="pending",
).select_related("course", "user")
```
