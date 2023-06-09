# Assignment
from flask import Flask, jsonify, request
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///tasks.db'
db = SQLAlchemy(app)


class Task(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(50), nullable=False)
    description = db.Column(db.String(200))
    due_date = db.Column(db.Date)
    status = db.Column(db.String(20), default='Incomplete')

    def to_dict(self):
        return {
            'id': self.id,
            'title': self.title,
            'description': self.description,
            'due_date': str(self.due_date),
            'status': self.status
        }


@app.route('/tasks', methods=['POST'])
def create_task():
    data = request.get_json()
    task = Task(title=data['title'], description=data['description'], due_date=data['due_date'])
    db.session.add(task)
    db.session.commit()
    return jsonify({'message': 'Task created successfully.'})


@app.route('/tasks/<int:task_id>', methods=['GET'])
def get_task(task_id):
    task = Task.query.get(task_id)
    if task:
        return jsonify(task.to_dict())
    else:
        return jsonify({'message': 'Task not found.'}), 404


@app.route('/tasks/<int:task_id>', methods=['PUT'])
def update_task(task_id):
    task = Task.query.get(task_id)
    if task:
        data = request.get_json()
        task.title = data['title']
        task.description = data['description']
        task.due_date = data['due_date']
        task.status = data['status']
        db.session.commit()
        return jsonify({'message': 'Task updated successfully.'})
    else:
        return jsonify({'message': 'Task not found.'}), 404


@app.route('/tasks/<int:task_id>', methods=['DELETE'])
def delete_task(task_id):
    task = Task.query.get(task_id)
    if task:
        db.session.delete(task)
        db.session.commit()
        return jsonify({'message': 'Task deleted successfully.'})
    else:
        return jsonify({'message': 'Task not found.'}), 404


@app.route('/tasks', methods=['GET'])
def list_tasks():
    page = request.args.get('page', default=1, type=int)
    per_page = request.args.get('per_page', default=10, type=int)
    tasks = Task.query.paginate(page=page, per_page=per_page)
    task_list = [task.to_dict() for task in tasks.items]
    return jsonify(task_list)


if __name__ == '__main__':
    db.create_all()
    app.run()
