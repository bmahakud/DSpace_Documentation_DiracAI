package org.dspace.app.rest.diracai.controller;

import org.dspace.app.rest.diracai.Repository.RoleAuditLogRepository;
import org.dspace.content.Diracai.RoleAuditLog;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/api/role-audit")
public class RoleAuditLogController {

    @Autowired
    private RoleAuditLogRepository repository;

    @GetMapping
    public List<RoleAuditLog> allLogs() {
        return repository.findAll();
    }
}

package org.dspace.app.rest.diracai.Repository;


import org.dspace.content.Diracai.RoleAuditLog;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;
import java.util.UUID;

public interface RoleAuditLogRepository extends JpaRepository<RoleAuditLog, UUID> {
    List<RoleAuditLog> findByAffectedUser(UUID affectedUser);
}

package org.dspace.content.Diracai;

import jakarta.persistence.*;
import java.sql.Timestamp;
import java.util.UUID;

@Entity
@Table(name = "role_audit_log")
public class RoleAuditLog {

    @Id
    @GeneratedValue
    private UUID id;

    @Column(name = "acted_by") // who performed the change
    private UUID actedBy;

    @Column(name = "affected_user")
    private UUID affectedUser;

    @Column(name = "action") // e.g., ROLE_ASSIGN, PERMISSION_GRANT
    private String action;

    @Column(name = "target") // target group/policy/item name/id
    private String target;

    @Column(name = "timestamp")
    private Timestamp timestamp;

    @Column(name = "ip_address")
    private String ipAddress;

    @Column(name = "user_agent", length = 512)
    private String userAgent;

    public void setUserAgent(String userAgent) {
        this.userAgent = userAgent;
    }

    public void setIpAddress(String ipAddress) {
        this.ipAddress = ipAddress;
    }

    public String getUserAgent() {
        return userAgent;
    }

    public UUID getId() {
        return id;
    }

    public String getIpAddress() {
        return ipAddress;
    }

    public void setId(UUID id) {
        this.id = id;
    }

    public String getAction() {
        return action;
    }

    public Timestamp getTimestamp() {
        return timestamp;
    }

    public String getTarget() {
        return target;
    }

    public UUID getActedBy() {
        return actedBy;
    }

    public UUID getAffectedUser() {
        return affectedUser;
    }

    public void setActedBy(UUID actedBy) {
        this.actedBy = actedBy;
    }

    public void setTarget(String target) {
        this.target = target;
    }

    public void setAction(String action) {
        this.action = action;
    }

    public void setAffectedUser(UUID affectedUser) {
        this.affectedUser = affectedUser;
    }

    public void setTimestamp(Timestamp timestamp) {
        this.timestamp = timestamp;
    }
}



package org.dspace.app.rest.diracai.service;

import jakarta.annotation.PostConstruct;
import jakarta.servlet.http.HttpServletRequest;
import org.dspace.app.rest.diracai.Repository.UserSessionAuditRepository;
import org.dspace.content.Diracai.UserSessionAudit;
import org.dspace.eperson.EPerson;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.sql.Timestamp;
import java.util.UUID;

@Component
public class UserSessionAuditService {

    private static UserSessionAuditService instance;

    @Autowired
    private UserSessionAuditRepository auditRepo;

    @PostConstruct
    public void registerInstance() {
        instance = this;
    }

    public static UserSessionAuditService getInstance() {
        return instance;
    }

    public void log(String eventType, EPerson user, HttpServletRequest request) {
        UserSessionAudit audit = new UserSessionAudit();
        audit.setId(UUID.randomUUID());
        audit.setUserId(UUID.fromString(user.getID().toString()));
        audit.setEmail(user.getEmail());
        audit.setEventType(eventType);
        audit.setLoginTime((new Timestamp(System.currentTimeMillis())));
        audit.setIpAddress(request.getRemoteAddr());
        audit.setUserAgent(request.getHeader("User-Agent"));
        audit.setTimestamp(new Timestamp(System.currentTimeMillis()));
        auditRepo.save(audit);
    }

    public void logLogout(EPerson user, HttpServletRequest request, String sessionId) {

        UserSessionAudit audit1 = auditRepo.findTopByUserIdAndIpAddressOrderByTimestampDesc(user.getID(), request.getRemoteAddr()).get();
        Timestamp logoutTime = new Timestamp(System.currentTimeMillis());
        audit1.setLogoutTime(logoutTime);
        if (audit1.getLoginTime() != null) {
            long duration = (logoutTime.getTime() - audit1.getLoginTime().getTime()) / 1000;
            audit1.setDurationInSeconds(duration);
        }
        audit1.setEventType("LOGOUT");
        audit1.setTimestamp(new Timestamp(System.currentTimeMillis()));
        auditRepo.save(audit1);
    }
}


package org.dspace.app.rest.diracai.Repository;

import org.dspace.content.Diracai.UserSessionAudit;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import java.util.Optional;

import java.util.List;
import java.util.UUID;

@Repository
public interface UserSessionAuditRepository extends JpaRepository<UserSessionAudit, UUID> {
    List<UserSessionAudit> findByUserId(UUID userId); // ✅ Add this line
    Optional<UserSessionAudit> findTopByUserIdAndIpAddressOrderByTimestampDesc(UUID userId, String ipAddress);

    Optional<UserSessionAudit> findTopByUserIdAndSessionIdAndEventTypeOrderByTimestampDesc(UUID userId, String sessionId, String eventType);

}

package org.dspace.content.Diracai;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

import java.sql.Timestamp;
import java.util.UUID;

@Entity
@Table(name = "user_session_audit")
public class UserSessionAudit {

    @Id
    @Column(columnDefinition = "uuid")
    private UUID id;

    @Column(name = "user_id", nullable = false)
    private UUID userId;

    @Column(name = "email")
    private String email;

    @Column(name = "ip_address")
    private String ipAddress;

    @Column(name = "user_agent")
    private String userAgent;

    @Column(name = "event_type")
    private String eventType; // LOGIN / LOGOUT

    @Column(name = "timestamp")
    private Timestamp timestamp;

    // 🔽 NEW FIELDS
    @Column(name = "session_id")
    private String sessionId;

    @Column(name = "login_time")
    private Timestamp loginTime;

    @Column(name = "logout_time")
    private Timestamp logoutTime;

    @Column(name = "duration_seconds")
    private Long durationInSeconds;

    public UserSessionAudit() {}

    // Getters and Setters

    public UUID getId() { return id; }
    public void setId(UUID id) { this.id = id; }

    public UUID getUserId() { return userId; }
    public void setUserId(UUID userId) { this.userId = userId; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public String getIpAddress() { return ipAddress; }
    public void setIpAddress(String ipAddress) { this.ipAddress = ipAddress; }

    public String getUserAgent() { return userAgent; }
    public void setUserAgent(String userAgent) { this.userAgent = userAgent; }

    public String getEventType() { return eventType; }
    public void setEventType(String eventType) { this.eventType = eventType; }

    public Timestamp getTimestamp() { return timestamp; }
    public void setTimestamp(Timestamp timestamp) { this.timestamp = timestamp; }

    public String getSessionId() { return sessionId; }
    public void setSessionId(String sessionId) { this.sessionId = sessionId; }

    public Timestamp getLoginTime() { return loginTime; }
    public void setLoginTime(Timestamp loginTime) { this.loginTime = loginTime; }

    public Timestamp getLogoutTime() { return logoutTime; }
    public void setLogoutTime(Timestamp logoutTime) { this.logoutTime = logoutTime; }

    public Long getDurationInSeconds() { return durationInSeconds; }
    public void setDurationInSeconds(Long durationInSeconds) { this.durationInSeconds = durationInSeconds; }
}


package org.dspace.app.rest.diracai.controller;

import org.dspace.app.rest.diracai.service.AuditService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

@RestController
@RequestMapping("/api/audit")
public class AuditController {

    @Autowired
    private AuditService auditService;

    @GetMapping("/users")
    public ResponseEntity<?> getAllUsers(){
        return auditService.getAllUsers();
    }

    @GetMapping("/user")
    public ResponseEntity<?> getByUser(@RequestParam("userId") UUID userId) {
        return auditService.getByUser(userId);
    }


}

package org.dspace.app.rest.diracai.service;

import org.springframework.http.ResponseEntity;

import java.util.UUID;

public interface AuditService {

    ResponseEntity<?> getAllUsers();

    ResponseEntity<?> getByUser(UUID userId);
}


package org.dspace.app.rest.diracai.service.impl;

import org.dspace.app.rest.diracai.Repository.FileAccessLogRepository;
import org.dspace.app.rest.diracai.Repository.RoleAuditLogRepository;
import org.dspace.app.rest.diracai.Repository.UserSessionAuditRepository;
import org.dspace.app.rest.diracai.dto.Audit;
import org.dspace.app.rest.diracai.dto.UserActionLog;
import org.dspace.app.rest.diracai.service.AuditService;
import org.dspace.content.Diracai.FileAccessLog;
import org.dspace.content.Diracai.RoleAuditLog;
import org.dspace.content.Diracai.UserSessionAudit;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;

import java.sql.Timestamp;
import java.util.*;

@Service
public class AuditServiceImpl implements AuditService {

    @Autowired
    private UserSessionAuditRepository userSessionAuditRepository;

    @Autowired
    private RoleAuditLogRepository roleAuditLogRepository;

    @Autowired
    private FileAccessLogRepository fileAccessLogRepository;


    @Override
    public ResponseEntity<?> getAllUsers() {
        List<UserSessionAudit> allAudits = userSessionAuditRepository.findAll();
        Map<UUID, UserSessionAudit> latestAuditMap = new HashMap<>();
        for (UserSessionAudit audit : allAudits) {
            UUID userId = audit.getUserId();
            if (!latestAuditMap.containsKey(userId)) {
                latestAuditMap.put(userId, audit);
            } else {
                Timestamp existingLoginTime = latestAuditMap.get(userId).getTimestamp();
                if (audit.getTimestamp() != null && audit.getTimestamp().after(existingLoginTime)) {
                    latestAuditMap.put(userId, audit);
                }
            }
        }
        List<Audit> auditList = new ArrayList<>();
        for (UserSessionAudit audit : latestAuditMap.values()) {
            Audit dto = new Audit();
            dto.setEmail(audit.getEmail());
            dto.setLoginTime(audit.getLoginTime());
            dto.setLogoutTime(audit.getLogoutTime());
            dto.setDuration(audit.getDurationInSeconds());
            dto.setUserId(audit.getUserId());
            auditList.add(dto);
        }
        return ResponseEntity.ok(auditList);
    }



    @Override
    public ResponseEntity<?> getByUser(UUID userId) {
        List<UserActionLog> actionLogs = new ArrayList<>();

        List<UserSessionAudit> sessionAudits = userSessionAuditRepository.findByUserId(userId);
        sessionAudits.stream()
                .filter(audit -> "LOGIN".equalsIgnoreCase(audit.getEventType()) || "LOGOUT".equalsIgnoreCase(audit.getEventType()))
                .forEach(audit -> {
                    UserActionLog log = new UserActionLog();
                    log.setAction(Optional.ofNullable(audit.getEventType()).orElse(null));
                    log.setTimestamp(Optional.ofNullable(audit.getTimestamp()).orElse(null));
                    log.setIpAddress(Optional.ofNullable(audit.getIpAddress()).orElse(null));
                    log.setUserAgent(Optional.ofNullable(audit.getUserAgent()).orElse(null));
                    actionLogs.add(log);
                });

        List<RoleAuditLog> auditLogs = roleAuditLogRepository.findByAffectedUser(userId);

        Optional<RoleAuditLog> latestRoleChange = auditLogs.stream()
                .filter(log -> "ROLE_ASSIGN".equalsIgnoreCase(log.getAction()))
                .max(Comparator.comparing(RoleAuditLog::getTimestamp, Comparator.nullsLast(Comparator.naturalOrder())));

        Optional<RoleAuditLog> latestPermissionChange = auditLogs.stream()
                .filter(log -> "PERMISSION_GRANT".equalsIgnoreCase(log.getAction()))
                .max(Comparator.comparing(RoleAuditLog::getTimestamp, Comparator.nullsLast(Comparator.naturalOrder())));

        UserActionLog roleLog = new UserActionLog();
        roleLog.setAction("ROLE_ASSIGN");
        roleLog.setTimestamp(latestRoleChange.map(RoleAuditLog::getTimestamp).orElse(null));
        roleLog.setIpAddress(latestRoleChange.map(RoleAuditLog::getIpAddress).orElse(null));
        roleLog.setUserAgent(latestRoleChange.map(RoleAuditLog::getUserAgent).orElse(null));
        actionLogs.add(roleLog);

        UserActionLog permissionLog = new UserActionLog();
        permissionLog.setAction("PERMISSION_GRANT");
        permissionLog.setTimestamp(latestPermissionChange.map(RoleAuditLog::getTimestamp).orElse(null));
        permissionLog.setIpAddress(latestPermissionChange.map(RoleAuditLog::getIpAddress).orElse(null));
        permissionLog.setUserAgent(latestPermissionChange.map(RoleAuditLog::getUserAgent).orElse(null));
        actionLogs.add(permissionLog);

        List<FileAccessLog> fileAccessLogs = fileAccessLogRepository.findByUserId(userId);


        Optional<FileAccessLog> latestDownload = fileAccessLogs.stream()
                .filter(log -> "DOWNLOAD".equalsIgnoreCase(log.getAction()))
                .max(Comparator.comparing(FileAccessLog::getTimestamp, Comparator.nullsLast(Comparator.naturalOrder())));

        if (latestDownload.isPresent()) {
            FileAccessLog log = latestDownload.get();
            UserActionLog downloadLog = new UserActionLog();
            downloadLog.setAction("DOWNLOAD");
            downloadLog.setTimestamp(log.getTimestamp());
            downloadLog.setIpAddress(log.getIpAddress());
            downloadLog.setUserAgent(log.getUserAgent());
            downloadLog.setObjectId(log.getFileName());
            actionLogs.add(downloadLog);
        }



        Optional<FileAccessLog> latestView = fileAccessLogs.stream()
                .filter(log -> "VIEW".equalsIgnoreCase(log.getAction()))
                .max(Comparator.comparing(FileAccessLog::getTimestamp, Comparator.nullsLast(Comparator.naturalOrder())));

        if (latestView.isPresent()) {
            FileAccessLog log = latestView.get();
            UserActionLog viewLog = new UserActionLog();
            viewLog.setAction("VIEW");
            viewLog.setTimestamp(log.getTimestamp());
            viewLog.setIpAddress(log.getIpAddress());
            viewLog.setUserAgent(log.getUserAgent());
            viewLog.setObjectId(log.getFileName());
            actionLogs.add(viewLog);
        }



        actionLogs.sort(Comparator.comparing(UserActionLog::getTimestamp, Comparator.nullsLast(Comparator.naturalOrder())));
        return ResponseEntity.ok(actionLogs);
    }

}


package org.dspace.app.rest.diracai.Repository;

import org.dspace.content.Diracai.FileAccessLog;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.UUID;

@Repository
public interface FileAccessLogRepository extends JpaRepository<FileAccessLog, UUID> {
    List<FileAccessLog> findByUserId(UUID userId);
    List<FileAccessLog> findByFileId(UUID fileId);
}

package org.dspace.content.Diracai;

import jakarta.persistence.*;

import java.sql.Timestamp;
import java.util.Date;
import java.util.UUID;

@Entity
@Table(name = "file_access_log")
public class FileAccessLog {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String fileName;
    private String action;
    private UUID userId;
    private UUID fileId;
    private String userEmail;
    private String ipAddress;
    private String userAgent;

    @Temporal(TemporalType.TIMESTAMP)
    private Timestamp timestamp;

    private boolean suspicious;

    public UUID getFileId() {
        return fileId;
    }

    public void setFileId(UUID fileId) {
        this.fileId = fileId;
    }

    public UUID getUserId() {
        return userId;
    }

    public void setUserId(UUID userId) {
        this.userId = userId;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getFileName() {
        return fileName;
    }

    public void setFileName(String fileName) {
        this.fileName = fileName;
    }

    public String getAction() {
        return action;
    }

    public void setAction(String action) {
        this.action = action;
    }

    public String getUserEmail() {
        return userEmail;
    }

    public void setUserEmail(String userEmail) {
        this.userEmail = userEmail;
    }

    public String getIpAddress() {
        return ipAddress;
    }

    public void setIpAddress(String ipAddress) {
        this.ipAddress = ipAddress;
    }

    public String getUserAgent() {
        return userAgent;
    }

    public void setUserAgent(String userAgent) {
        this.userAgent = userAgent;
    }

    public Timestamp getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(Timestamp timestamp) {
        this.timestamp = timestamp;
    }

    public boolean isSuspicious() {
        return suspicious;
    }

    public void setSuspicious(boolean suspicious) {
        this.suspicious = suspicious;
    }
}


package org.dspace.content.Diracai;

import jakarta.persistence.*;
import java.util.UUID;
import java.sql.Timestamp;

@Entity
@Table(name = "login_device_audit")
public class LoginDeviceAudit {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private UUID id;

    @Column(name = "eperson_id", nullable = false)
    private UUID epersonUUID;

    @Column(name = "ip_address")
    private String ipAddress;

    @Column(name = "user_agent", columnDefinition = "TEXT")
    private String userAgent;

    @Column(name = "device_id")
    private String deviceId;

    @Column(name = "login_time")
    private Timestamp loginTime;

    @Column(name = "status")  // e.g. SUCCESS / FAILURE
    private String status;

    @Column(name = "failed_attempts")
    private int failedAttempts = 0;

    // Getters and Setters
    public UUID getId() {
        return id;
    }

    public void setId(UUID id) {
        this.id = id;
    }

    public UUID getEpersonUUID() {
        return epersonUUID;
    }

    public void setEpersonUUID(UUID epersonUUID) {
        this.epersonUUID = epersonUUID;
    }

    public String getIpAddress() {
        return ipAddress;
    }

    public void setIpAddress(String ipAddress) {
        this.ipAddress = ipAddress;
    }

    public String getUserAgent() {
        return userAgent;
    }

    public void setUserAgent(String userAgent) {
        this.userAgent = userAgent;
    }

    public String getDeviceId() {
        return deviceId;
    }

    public void setDeviceId(String deviceId) {
        this.deviceId = deviceId;
    }

    public Timestamp getLoginTime() {
        return loginTime;
    }

    public void setLoginTime(Timestamp loginTime) {
        this.loginTime = loginTime;
    }

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }

    public int getFailedAttempts() {
        return failedAttempts;
    }

    public void setFailedAttempts(int failedAttempts) {
        this.failedAttempts = failedAttempts;
    }
}



