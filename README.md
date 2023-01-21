# CAR MANAGEMENT SYSTEM
#### Video Demo:  <https://clipchamp.com/watch/KWq8pKOYz4V>
#### Description: **Management application for car repair process**


>Hello, world!
My name is Yuri Petrillo and this is my project, i made it because i hope it will become really useful, when i was thinking to create it i was thinking about many realistic use cases, and I will certainly make many other improvements.
It was born with the idea of managing the repairs of an auto repair company.

I have used external sources via API to retrieve all the car models for each single year of construction in a consistent and updated manner, as per below.
### External API
```
https://www.fueleconomy.gov
```

For this project i've built a web application that uses React for the frontend with LocalStorage, React Router, Axios and Bootstrap.
For secure the application i've used JWT Authentication so any Client must add JWT token to HTTP Header before can be able to send request to protected resources of Backend.

### Ports configuration
```
FE PORT=8082
BE PORT=8080
```

To register a new user and give authentication we will proceed in this way:

  - POST call to  */signup*  API with username, password and email, Check input data and give response back<br />
  - POST call to  */login*  API with personal username and password and get back new generated Token ( *JWT* ), user type and authorities<br />
  - POST call with Token on Request Header and check for user authorities and then give access to correct resources.<br />

FE will use AXIOS to make requests and Local Storage to store or get jwt token from local browser and react validation to validate input data from user.

This Application is developed as Microservice Architecture and every REST API Controller can communicate with database and apply business logic using dedicated services.
<br /><br />

**Controllers**: 
- *AuthController*
- *CarController*
- *CustomerController*
- *EmployeeController*
- *RepairController*
- *UserController*
- *WorkController*

**Services**:
- *UserDetailService*
- *CarService*
- *CustomerService*
- *EmployeeService*
- *RepairService*
- *WorkService*


Each entity has own repository and it's configured with required relationship as shown in below diagram.<br />
![Relational Database Schema Detail](CMS_Schema.jpg)

Here an example of Rest endpoint controller responsible to give repair by employee (this endpoint can be called also with USER role):
```
@GetMapping("/getRepairEmployee/{employeeId}")
  @PreAuthorize("hasRole('SUPERVISOR') or hasRole('ADMIN') or hasRole('USER')")
  public ResponseEntity<?> getRepairsByEmployeeId(@PathVariable Long employeeId) throws RepairException {
	  List<Repair> actualRepairs = repairService.getRepairsByEmployeeId(employeeId);
    return ResponseEntity.ok(actualRepairs);
  }
```

Here instead an example of Service responsible to intermediate and give back repairs for certain employee but only uncompleted ones:
```
	public List<Repair> getRepairsByEmployeeIdAndFilterWorkingDone(Long employeeId) throws RepairException {
		return repairRepository.findByEmployee_employeeIdAndIsWorkingFalseAndIsDoneFalse(employeeId).orElseThrow(() -> new RepairException("Repairs not done of employee " +employeeId+" Not FOUND!"));
	}
```

Custom configuration for FilterChain:
```
@Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.cors().and().csrf().disable()
        .exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()
        .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
        .authorizeRequests()
        .antMatchers("/api/auth/**").permitAll()
        .antMatchers("/api/verify/**").permitAll()
        .anyRequest().authenticated();
    
    http.authenticationProvider(authenticationProvider());
    http.addFilterBefore(authenticationJwtTokenFilter(), UsernamePasswordAuthenticationFilter.class);
    
    return http.build();
  }
```
