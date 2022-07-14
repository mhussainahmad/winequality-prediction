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