create env

'''bash
conda create -n wineq python=3.7 -y
'''

activate env
'''bash
conda activate wineq
'''

create a requirements.txt file

install the erquirements
'''bash
pip install -r requirements.txt
'''

download the dataset 

git init 
dvc init
dvc add data_given/winequality.csv
git add .
git commit -m "first commit"


create an artifcats folder

mlflow server command -

mlflow server \
    --backend-store-uri sqlite:///mlflow.db \
    --default-artifact-root ./artifacts \
    --host localhost -p 5000