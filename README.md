"# Week_14" 
[Untitled document (2).pdf](https://github.com/Jballeno/Week_14/files/12271478/Untitled.document.2.pdf)

https://drive.google.com/file/d/1Kr-VHlXRtIDSlG6-eGjb3sST71sKBaGh/view?usp=sharing
package pet.park.controller.model;
import java.util.HashSet;
import java.util.Set;
import lombok.Data;
import lombok.NoArgsConstructor;
import pet.park.entity.Amenity;
import pet.park.entity.Contributor;
import pet.park.entity.GeoLocation;
import pet.park.entity.PetPark;
@Data
@NoArgsConstructor
public class PetParkData {
private Long petParkId;
private String parkName;
private String directions;
private String stateOrProvince;
private String country;
private GeoLocation geoLocation;
private PetParkContributor contributor;
private Set<String> amenities = new HashSet<>();
public PetParkData(PetPark petPark) {
petParkId = petPark.getPetParkId();
parkName = petPark.getParkName();
directions = petPark.getDirections();
stateOrProvince = petPark.getStateOrProvince();
country = petPark.getCountry();
geoLocation = petPark.getGeoLocation();
contributor = new PetParkContributor(petPark.getContributor());
for(Amenity amenity : petPark.getAmenities()) {
amenities.add(amenity.getAmenity());
}
}
@Data
@NoArgsConstructor
public static class PetParkContributor {
private Long contributorId;
private String contributorName;
private String contributorEmail;
public PetParkContributor(Contributor contributor) {
contributorId = contributor.getContributorId();
contributorName = contributor.getContributorName();
contributorEmail = contributor.getContributorEmail();
}
}
}
package pet.park.dao;
import org.springframework.data.jpa.repository.JpaRepository;
import pet.park.entity.PetPark;
public interface PetParkDao extends JpaRepository<PetPark, Long> {
}
package pet.park.service;
import java.util.LinkedList;
import java.util.List;
import java.util.NoSuchElementException;
import java.util.Objects;
import java.util.Optional;
import java.util.Set;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.dao.DuplicateKeyException;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import pet.park.controller.model.ContributorData;
import pet.park.controller.model.PetParkData;
import pet.park.dao.AmenityDao;
import pet.park.dao.ContributorDao;
import pet.park.dao.PetParkDao;
import pet.park.entity.Amenity;
import pet.park.entity.Contributor;
import pet.park.entity.PetPark;
@Service
public class ParkService {
@Autowired
private ContributorDao contributorDao;
@Autowired
private AmenityDao amenityDao;
@Autowired
private PetParkDao petParkDao;
@Transactional(readOnly = false)
public ContributorData saveContributor(ContributorData contributorData) {
Long contributorId = contributorData.getContribtorId();
Contributor contributor = findOrCreateContributor(contributorId,
contributorData.getContributorEmail());
setFieldsContributor(contributor, contributorData);
return new ContributorData(contributorDao.save(contributor));
}
private void setFieldsContributor(Contributor contributor, ContributorData
contributorData) {
contributor.setContributorEmail(contributorData.getContributorEmail());
contributor.setContributorName(contributorData.getContributorName());
}
private Contributor findOrCreateContributor(Long contributorId, String contributorEmail) {
Contributor contributor;
if (Objects.isNull(contributorId)) {
Optional<Contributor> opContrib =
contributorDao.findByContributorEmail(contributorEmail);
if(opContrib.isPresent()) {
throw new DuplicateKeyException(
"Contribtor with email " + contributorEmail + "
already exists");
}
contributor = new Contributor();
} else {
contributor = findContributorById(contributorId);
}
return contributor;
}
private Contributor findContributorById(Long contributorId) {
return contributorDao.findById(contributorId).orElseThrow(
() -> new NoSuchElementException("Contributor with ID=" +
contributorId + " was not found."));
}
@Transactional(readOnly = true)
public List<ContributorData> retrieveAllContributors() {
List<Contributor> contributors = contributorDao.findAll();
List<ContributorData> response = new LinkedList<>();
for (Contributor contributor : contributors) {
response.add(new ContributorData(contributor));
}
return response;
}
@Transactional(readOnly = true)
public ContributorData retrieveContributorById(Long contributorId) {
Contributor contributor = findContributorById(contributorId);
return new ContributorData(contributor);
}
@Transactional(readOnly = false)
public void deleteContributorById(Long contributorId) {
Contributor contributor = findContributorById(contributorId);
contributorDao.delete(contributor);
}
@Transactional(readOnly = false)
public PetParkData savePetPark(Long contributorId,
PetParkData petParkData) {
Contributor contributor = findContributorById(contributorId);
Set<Amenity> amenities =
amenityDao.findAllByAmenityIn(petParkData.getAmenities());
PetPark petPark = findOrCreatePetPark(petParkData.getPetParkId());
setPetParkFields(petPark, petParkData);
petPark.setContributor(contributor);
contributor.getPetParks().add(petPark);
for(Amenity amenity : amenities) {
amenity.getPetParks().add(petPark);
petPark.getAmenities().add(amenity);
}
PetPark dbPetPark = petParkDao.save(petPark);
return new PetParkData(dbPetPark);
}
private void setPetParkFields(PetPark petPark, PetParkData petParkData) {
petPark.setCountry(petParkData.getCountry());
petPark.setDirections(petParkData.getDirections());
petPark.setGeoLocation(petParkData.getGeoLocation());
petPark.setParkName(petParkData.getParkName());
petPark.setStateOrProvince(petParkData.getStateOrProvince());
}
private PetPark findOrCreatePetPark(Long petParkId) {
PetPark petPark;
if(Objects.isNull(petParkId)) {
petPark = new PetPark();
}
else {
petPark = findPetParkById(petParkId);
}
return petPark;
}
private PetPark findPetParkById(Long petParkId) {
return petParkDao.findById(petParkId).orElseThrow(() -> new
NoSuchElementException("Pet Park with ID=" + petParkId + " does not exsist."));
}
@Transactional (readOnly = true)
public PetParkData retrievePetParkById(Long contributorId, Long parkId) {
findContributorById(contributorId);
PetPark petPark = findPetParkById(parkId);
if(petPark.getContributor().getContributorId() != contributorId) {
throw new IllegalStateException("Pet park with ID=" + parkId + " is not
owned by contributor with ID=" + contributorId);
}
return new PetParkData(petPark);
}
}
package pet.park.controller.error;
import java.time.ZonedDateTime;
import java.time.format.DateTimeFormatter;
import java.util.NoSuchElementException;
import org.springframework.dao.DuplicateKeyException;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.context.request.ServletWebRequest;
import org.springframework.web.context.request.WebRequest;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
@RestControllerAdvice
@Slf4j
public class GlobaErrorHandler {
private enum Logstatus {
STACK_TRACE, MESSAGE_ONLY
}
@Data
private class ExceptionMessage {
private String message;
private String StatusReason;
private int statusCode;
private String TimeStamp;
private String uri;
}
@ExceptionHandler(IllegalStateException.class)
@ResponseStatus(code = HttpStatus.BAD_REQUEST)
public ExceptionMessage handleillExceptionMessage(IllegalStateException ex,
WebRequest webRequest) {
return buildExceptionMessage(ex, HttpStatus.BAD_REQUEST, webRequest,
Logstatus.MESSAGE_ONLY);
}
@ExceptionHandler(UnsupportedOperationException.class)
@ResponseStatus(code = HttpStatus.METHOD_NOT_ALLOWED)
public ExceptionMessage handleUnsupportedOperationException(
UnsupportedOperationException ex, WebRequest webRequest) {
return buildExceptionMessage(ex, HttpStatus.METHOD_NOT_ALLOWED,
webRequest, Logstatus.MESSAGE_ONLY);
}
@ExceptionHandler(NoSuchElementException.class)
@ResponseStatus(code = HttpStatus.NOT_FOUND)
public ExceptionMessage handleNoSuchElementExpetion (
NoSuchElementException ex, WebRequest webRequest) {
return buildExceptionMessage(ex, HttpStatus.NOT_FOUND, webRequest,
Logstatus.MESSAGE_ONLY);
}
@ExceptionHandler(DuplicateKeyException.class)
@ResponseStatus(code = HttpStatus.CONFLICT)
public ExceptionMessage handleDuplicateExceptionMessage(DuplicateKeyException
ex, WebRequest webRequest) {
return buildExceptionMessage(ex, HttpStatus.CONFLICT, webRequest,
Logstatus.MESSAGE_ONLY);
}
@ExceptionHandler(Exception.class)
@ResponseStatus(code = HttpStatus.INTERNAL_SERVER_ERROR)
public ExceptionMessage handleException(Exception ex, WebRequest webRequest) {
return buildExceptionMessage(ex, HttpStatus.INTERNAL_SERVER_ERROR,
webRequest, Logstatus.STACK_TRACE);
}
private ExceptionMessage buildExceptionMessage(Exception ex, HttpStatus status,
WebRequest webRequest,
Logstatus logStatus) {
String message = ex.toString();
String statusReason = status.getReasonPhrase();
int statusCode = status.value();
String uri = null;
String TimeStamp =
ZonedDateTime.now().format(DateTimeFormatter.RFC_1123_DATE_TIME);
if(webRequest instanceof ServletWebRequest swr) {
uri = swr.getRequest().getRequestURI();
}
if(logStatus == Logstatus.MESSAGE_ONLY) {
log.error("Excption: (),", ex.toString());
}
else {
log.error("Exception: ", ex);
}
ExceptionMessage exMsg = new ExceptionMessage();
exMsg.setMessage(message);
exMsg.setStatusCode(statusCode);
exMsg.setStatusReason(statusReason);
exMsg.setTimeStamp(TimeStamp);
exMsg.setUri(uri);
return exMsg;
}
}
