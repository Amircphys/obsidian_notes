```
from clearml import Task
task = Task.init(
	project_name='my_project', 
	task_name='my_task'
	task_type="my_task_type",
	reuse_last_task_id=False,
)
# Log and articfact
task.upload_artifact(...)

# Log an image
task.get_logger().report_image(...)

# Log a video or audio
task.get_logger().report_media(...)

# add simple hyperparameters
params = {"lr": 0.001, "alpha": 1.2}
task.connect(params)

# add configuration object
config_file_yaml = task.connect_configuration(
 name="yaml file",
 configuration="config.yaml",
)
# Можно клонировать эксперимент, изменить какой-то параметр и запустит #эксперимент прямо в UI


# artifact - это можеть быть преобработанные данные, модель и тд
# артефакты из одного эксперимента можно использовать в других
taks.upload_artifact(...)
t = Task.get_task(taks_id=1)
a = t.artifacts['features']
x = a.get_local_copy()

# add tag to task:
taks.add_tags(["new_tag_1", "new_tag_2", ...])
```
## ClearML Workflow
![[Pasted image 20241103150511.png]]Если используется tesnorboard - clearml автоматически подхватывает логи, которые идут в  tesnorboard

Если используется matplotlib - графики автоматически подхватываются clearml