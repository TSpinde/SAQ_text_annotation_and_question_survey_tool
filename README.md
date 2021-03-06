# survey-spa-flask-vue
A survey platform built on Flask and Vue.js. This survey app deals with basic questions such as *DataConsent*, *Demographics*, *Ideologies* and *QualityControl* and text highlighting question where the participants can annotate words/phrases wthat show certain properties. 

## To import the project locally

1) Make sure you have all the dependencies resolved in requirements.txt.
2) The current version of the app runs on a WSGI server hosted on pythonanywhere.com. To run the project on a locally, please create a local flask server.
3) Next, please add this line `app.run(0.0.0.0)` at the end of the file `flask_app.py`.
4) Configure a local MySQL server or a SQLLite server and make the appropriate changes in `surveyapi.models.db` to get the DB up and running.

## Platform structure and hierarchies

- Models for simple survey response mechanisms such as simple single select choice, multiple select, chips select, range sliders and text highlighter already exist in `surveyapi.models`
- To create new response model, please do the appropriate DB migrations as per the flask and your DB configurations.
- The following main models (and corresponding tables) exist in the survey and can be used out of the box:
> 1. **SurveyRecord**: Holds the parent object generated from the responses such as such as *DataConsent*, *Demographics*, *Ideologies* and *QualityControl* from a participant.
> 2. **SurveyGroups**: Groups the sentences that are to be annotated. Also holds the max quota variable of the groups.
> 3. **Survey**: Encapsulator for various types of surveys such as demographics, ideologies etc.
> 4. **Question**: Holds a survey question. Can be specified with the type of response the question should hold.
> 3. **AnnotationSentences**: Actual sentences that are to be annotated. Holds other propeties such as pulication, urls etc.
> 4. **Annotations**: Holds the words/phrases highlighted/annotated by the participants.

File seeder.py is used to seed and populate the platform database with questions etc.

## Creating a simple demographic question with single choice select

Run the file seeder.py with these lines of code.

```python
survey = Survey(name="demographic_questions")
questions = []

# 1
question_1 = Question(text='What is your gender?', type='radio', name='gender')
question_1.simple_choices = [
    SimpleChoice(text='Female'),
    SimpleChoice(text='Male'),
    SimpleChoice(text='Other/Prefer not to say')
]
questions.append(question_1)

survey.questions = questions
db.session.add(survey)
db.session.commit()
```


## Creating a simple ideological question with range slider select

Run the file seeder.py with these lines of code.

```python
survey = Survey(name="idealogy_questions")
idealogy_questions = []
question_1 = Question(
    text='Do you consider yourself to be liberal, conservative or somewhere in between?',
    type='range_slider',
    name='political_ideology'
)
question_1.range_slider_choices = [
    RangeSliderChoice(
        min_range=-10,
        max_range=10,
        label_left_side='Very liberal',
        label_right_side='Very conservative'
    )
]
idealogy_questions.append(question_1)
survey.questions = idealogy_questions
db.session.add(survey)
db.session.commit()
```

## Seeding annotation sentences

In the file seeder.py, use the function `seed_annotation_sentences()` to do so.

```python
    app = create_app()
    ctx = app.app_context()
    ctx.push()
    db.init_app(app)
    with open('sentences_grouped_reshuffled.csv') as f:
        reader = csv.DictReader(f)
        for row in reader:
            annotation_sentence = AnnotationSentences(
                id=uuid.uuid4().hex,
                text=row['sentence'],
                link=row['news_link'],
                type=row['type'],
                topic=row['topic'],
                outlet=row['outlet'],
                group_id=int(row['group_id'])
            )
            db.session.add(annotation_sentence)
            db.session.commit()
    ctx.pop()

seed_annotation_sentences()
```

The function expects a CSV file such as the following `https://github.com/kanishk2509/survey-spa-flask-vue/blob/master/sentences_grouped_reshuffled.csv`

## Seeding survey questions

In the file seeder.py, use the functions 

1. `seed_personal_questions()`
2. `seed_idealogy_questions()`
3. `seed_annotation_questions()`
4. `create_groups()`


## Extracting the survey output

The output is extracted in the CSV format. Use the below command to open the interactive python shell.

`python -i db_helper.py`

Get the db 
`db, ctx = create_db_client()`

Get all the survey worker records as CSV
`to_csv_survey_worker_records(db, ctx, after_date=optional)`

Get all the survey annotations as CSV
`to_csv_all_annotations(db, ctx, after_date=optional)`

## In action

https://unisurveyapp.pythonanywhere.com/
