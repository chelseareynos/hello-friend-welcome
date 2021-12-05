queue: 'Hosted Linux Preview'

 steps:
 - task: UsePythonVersion@0
   inputs:
     versionSpec: 3.5
     architecture: 'x64'

 - task: PythonScript@0
   displayName: 'Export project path'
   inputs:
     scriptSource: 'inline'
     script: |
       """Search all subdirectories for `manage.py`."""
       from glob import iglob
       from os import path
       # Python >= 3.5
       manage_py = next(iglob(path.join('**', 'manage.py'), recursive=True), None)
       if not manage_py:
           raise SystemExit('Could not find a Django project')
       project_location = path.dirname(path.abspath(manage_py))
       print('Found Django project in', project_location)
       print('##vso[task.setvariable variable=projectRoot]{}'.format(project_location))

 - script: |
     python -m pip install --upgrade pip setuptools wheel django
     pip install -r requirements.txt
     pip install unittest-xml-reporting
   displayName: 'Install prerequisites'

 - script: |
     pushd '$(projectRoot)'
     python manage.py test --testrunner xmlrunner.extra.djangotestrunner.XMLTestRunner --no-input
   condition: succeededOrFailed()
   displayName: 'Run tests'

 - task: PublishTestResults@2
   inputs:
     testResultsFiles: "**/TEST-*.xml"
     testRunTitle: 'Python $(PYTHON_VERSION)'
