from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///students.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

# Student Model
class Student(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    age = db.Column(db.Integer, nullable=False)
    grade_level = db.Column(db.String(50), nullable=False)
    parent_name = db.Column(db.String(100), nullable=False)
    parent_contact = db.Column(db.String(15), nullable=False)
    address = db.Column(db.String(200), nullable=False)

# Create the database
with app.app_context():
    db.create_all()

# Get all students
@app.route('/students', methods=['GET'])
def get_students():
    students = Student.query.all()
    return jsonify([[student.id, student.name, student.age, student.grade_level, student.parent_name, student.parent_contact, student.address] for student in students])

# Get a single student
@app.route('/students/<int:id>', methods=['GET'])
def get_student(id):
    student = Student.query.get_or_404(id)
    return jsonify([student.id, student.name, student.age, student.grade_level, student.parent_name, student.parent_contact, student.address])

# Add a new student
@app.route('/students', methods=['POST'])
def add_student():
    data = request.get_json()
    new_student = Student(
        name=data['name'],
        age=data['age'],
        grade_level=data['grade_level'],
        parent_name=data['parent_name'],
        parent_contact=data['parent_contact'],
        address=data['address']
    )
    db.session.add(new_student)
    db.session.commit()
    return jsonify({'message': 'Student added successfully!'}), 201

# Update a student
@app.route('/students/<int:id>', methods=['PUT'])
def update_student(id):
    data = request.get_json()
    student = Student.query.get_or_404(id)
    student.name = data['name']
    student.age = data['age']
    student.grade_level = data['grade_level']
    student.parent_name = data['parent_name']
    student.parent_contact = data['parent_contact']
    student.address = data['address']
    db.session.commit()
    return jsonify({'message': 'Student updated successfully!'})

# Delete a student
@app.route('/students/<int:id>', methods=['DELETE'])
def delete_student(id):
    student = Student.query.get_or_404(id)
    db.session.delete(student)
    db.session.commit()
    return jsonify({'message': 'Student deleted successfully!'})

if __name__ == '__main__':
    app.run(debug=True)
