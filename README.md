# search-by name, keyword, partial keyword


#### `employee-bd.service.ts`
```bash
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Employee } from './employees.entity';
import { ILike, Like, Repository } from 'typeorm';

@Injectable()
export class EmployeeBdService {
    constructor(
        @InjectRepository(Employee) private employeeRepository: Repository<Employee>,
    ) {}

    async create(employeeData: Partial<Employee>): Promise<Employee> {
        const employee = this.employeeRepository.create(employeeData);
        return this.employeeRepository.save(employee);
    }

    async findAll(): Promise<Employee[]> {
        return this.employeeRepository.find();
    }

    async findOne(id: number): Promise<Employee> {
        const employee = await this.employeeRepository.findOneBy({ id });
        if (!employee) {
            throw new NotFoundException(`Employee with ID ${id} not found`);
        }
        return employee;
    }

    async findByName(name: string): Promise<Employee[]> {
        const employee = await this.employeeRepository.find({
            where: { name },
        });
        if (!employee.length) {
            throw new NotFoundException(`Employee with name ${name} not found`);
        }
        return employee;
    }

    // Like is case-sensitive, ILike is case-insensitive
    async searchByName(keyword: string): Promise<Employee[]> {
        return this.employeeRepository.find({
            where: {
                //name: Like(`%${keyword}%`)
                name: ILike(`%${keyword}%`)
            }
        });
    }

}
```

#

#### `employee-bd.controller.ts`
```bash
import { Body, Controller, Get, Param, Post } from '@nestjs/common';
import { EmployeeBdService } from './employee-bd.service';
import { Employee } from './employees.entity';

@Controller('employee-bd')
export class EmployeeBdController {
    constructor(private readonly employeeBdService: EmployeeBdService) {}

    @Post()
    async createEmployee(@Body() employeeData: Partial<Employee>) {
        return this.employeeBdService.create(employeeData);
    }

    @Get()
    async findAllEmployees(): Promise<Employee[]> {
        return this.employeeBdService.findAll();
    }

    // find by id
    @Get(':id')
    async findEmployeeById(@Param('id') id: number): Promise<Employee> {
        return this.employeeBdService.findOne(id);
    }

    // find by name
    @Get('name/:name')
    async findEmployeeByName(@Param('name') name: string): Promise<Employee[]> {
        return this.employeeBdService.findByName(name);
    }

    // search by name (partial match)
    @Get(`search/:keyword`)
    async searchEmployeeByName(@Param('keyword') keyword: string): Promise<Employee[]> {
        return this.employeeBdService.searchByName(keyword);
    }

}
```
---
