## Admin-Pool Feature

The **Admin-Pool** feature is designed to manage batch files and control their approval workflow based on user roles.

### Key Features

- When a new batch is uploaded, its details appear in the pool.
- Users with appropriate rights and permissions can **approve** or **reject** a batch file.
- Once a batch is approved or rejected, it moves to its respective pool.
- Each batch file in the pool shows details such as the uploader and the associated collection.
- Clicking **Perform Task** under a batch file displays detailed item information at the bottom.
- Actions performed on batch files are reflected in the respective pools.
- **Role-based view:**
  - **Admin:** Can see all approved or rejected files and all user actions.
  - **Reviewer:** Can see only the batch files they have claimed.
  - **Uploader:** Can only upload batch files.

### Role Scopes

| Role     | Scope of Actions                            |
|----------|---------------------------------------------|
| Admin    | Upload and review any batch file            |
| Uploader | Upload batch files only                     |
| Reviewer | Review batch files only                     |



You can follow this code for frontend implementation







// dspace_frontend_latest-main/src/app/app-routes.ts



import type { InMemoryScrollingOptions, Route, RouterConfigOptions } from "@angular/router"

import {
  ERROR_PAGE,
  INTERNAL_SERVER_ERROR,
} from "./app-routing-paths"
import { authBlockingGuard } from "./core/auth/auth-blocking.guard"
import { authenticatedGuard } from "./core/auth/authenticated.guard"
import { endUserAgreementCurrentUserGuard } from "./core/end-user-agreement/end-user-agreement-current-user.guard"
import { ServerCheckGuard } from "./core/server-check/server-check.guard"
import { menuResolver } from "./menuResolver"
import { provideSuggestionNotificationsState } from "./notifications/provide-suggestion-notifications-state"
import { ThemedPageErrorComponent } from "./page-error/themed-page-error.component"
import { ThemedPageInternalServerErrorComponent } from "./page-internal-server-error/themed-page-internal-server-error.component"
// remaining imports


export const APP_ROUTES: Route[] = [
  { path: INTERNAL_SERVER_ERROR, component: ThemedPageInternalServerErrorComponent },
  { path: ERROR_PAGE, component: ThemedPageErrorComponent },
  {
    path: "",
    canActivate: [authBlockingGuard],
    canActivateChild: [ServerCheckGuard],
    resolve: [menuResolver],
    children: [
      { path: "", redirectTo: "/home", pathMatch: "full" },
      {
        path: "adminpool",
        loadChildren: () => import("./admin-pool/admin-pool.module").then((m) => m.AdminPoolModule),
        data: { enableRSS: true },
        providers: [provideSuggestionNotificationsState()],
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      }

      // remaining paths ....

    ]
  },
]
export const APP_ROUTING_CONF: RouterConfigOptions = {
  onSameUrlNavigation: "reload",
}
export const APP_ROUTING_SCROLL_CONF: InMemoryScrollingOptions = {
  scrollPositionRestoration: "top",
  anchorScrolling: "enabled",
}









you can follow this code for backend implementation




// dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/controller/BulkUploadRequestRestController.java




package org.dspace.app.rest.diracai.controller;

import jakarta.servlet.http.HttpServletRequest;
import org.dspace.app.rest.diracai.dto.AuthTokenPayload;
import org.dspace.app.rest.diracai.dto.BulkFileDto;
import org.dspace.app.rest.diracai.dto.BulkUploadRequestResponseDTO;
import org.dspace.app.rest.diracai.service.BulkUploadRequestService;
import org.dspace.app.rest.diracai.service.impl.BulkUploadRequestServiceImpl;
import org.dspace.app.rest.utils.ContextUtil;
import org.dspace.content.Diracai.BulkUploadRequest;
import org.dspace.core.Context;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.util.List;
import java.util.Map;
import java.util.UUID;

@RestController
@RequestMapping("/api/bulk-upload")
public class BulkUploadRequestRestController {

    @Autowired
    private BulkUploadRequestService service;

    private static final Logger logger = LoggerFactory.getLogger(BulkUploadRequestRestController.class);

    @PostMapping("/upload/{uuid}")
    public ResponseEntity<BulkUploadRequest> upload(HttpServletRequest httpRequest,
                                                    @RequestParam("file") MultipartFile file,
                                                    @PathVariable UUID uuid) {
        try {
            Context context = ContextUtil.obtainContext(httpRequest);

            logger.info("📎 File received: name='{}', size={} bytes", file.getOriginalFilename(), file.getSize());

            BulkUploadRequest request = service.createRequest(context, file, uuid);

            logger.info("✅ Upload successful for BulkUploadRequest UUID: {}", request.getBulkUploadId());
            return ResponseEntity.ok(request);

        } catch (Exception e) {
            logger.error("❌ Failed to upload bulk file for UUID: {} — {}", uuid, e.getMessage(), e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }



    @PostMapping("/approve/{uuid}")
    public ResponseEntity<?> approve(@PathVariable UUID uuid, HttpServletRequest httpRequest) {
        try {
            Context context = ContextUtil.obtainContext(httpRequest);
            String authHeader = httpRequest.getHeader("Authorization");
            String csrfToken = httpRequest.getHeader("X-CSRF-TOKEN");

            AuthTokenPayload auth = new AuthTokenPayload();
            auth.setAuthorization(authHeader);
            auth.setCsrfToken(csrfToken);

            BulkUploadRequest result = service.approveRequest(uuid, context, auth);
            return ResponseEntity.ok(result);

        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.GONE)
                    .body(Map.of("error", e.getClass().getSimpleName(), "message", e.getMessage()));
        }
    }


    @PostMapping("/reject/{uuid}")
    public ResponseEntity<BulkUploadRequest> reject(@PathVariable UUID uuid, HttpServletRequest httpRequest) {
        Context context = ContextUtil.obtainContext(httpRequest);
        return ResponseEntity.ok(service.rejectRequest(uuid, context));
    }

    @GetMapping("/all")
    public ResponseEntity<List<BulkUploadRequest>> all() {
        return ResponseEntity.ok(service.findAll());
    }

    @GetMapping("/status/{status}")
    public ResponseEntity<List<BulkFileDto>> byStatus(@PathVariable String status, HttpServletRequest httpRequest) {
        Context context = ContextUtil.obtainContext(httpRequest);
        return ResponseEntity.ok(service.findByStatus(context,status));
    }

    @GetMapping("/pooled")
    public ResponseEntity<List<BulkFileDto>> getPooledTasks(HttpServletRequest httpRequest) {
        try {
            Context context = ContextUtil.obtainContext(httpRequest);

            UUID reviewerId = context.getCurrentUser().getID(); // Authenticated user

            List<BulkFileDto> pooledTasks = service.getPooledTasksForReviewer(context, reviewerId);
            return ResponseEntity.ok(pooledTasks);
        } catch (Exception e) {
            logger.error("❌ Failed to retrieve pooled tasks — {}", e.getMessage(), e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(null);
        }
    }

    @GetMapping("/{uuid}")
    public ResponseEntity<?> getFile(@PathVariable UUID uuid) {
        try {
            return ResponseEntity.ok(service.getFile(uuid));
        } catch (Exception e) {
            e.printStackTrace(); // Temporary for debugging
            return ResponseEntity.status(500).body(Map.of(
                    "message", "Failed to fetch bulk upload file",
                    "error", e.getMessage()
            ));
        }
    }

}




// dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/service/BulkUploadRequestService.java


package org.dspace.app.rest.diracai.service;

import org.dspace.app.rest.diracai.dto.AuthTokenPayload;
import org.dspace.app.rest.diracai.dto.BulkFileDto;
import org.dspace.app.rest.diracai.dto.BulkUploadRequestResponseDTO;
import org.dspace.content.Diracai.BulkUploadRequest;
import org.dspace.core.Context;
import org.springframework.web.multipart.MultipartFile;

import java.util.List;
import java.util.UUID;

public interface BulkUploadRequestService {
    BulkUploadRequest createRequest(Context context, MultipartFile file ,UUID uuid);
    BulkUploadRequest approveRequest(UUID uuid, Context context, AuthTokenPayload auth);
    BulkUploadRequest rejectRequest(UUID uuid, Context context);
    List<BulkUploadRequest> findAll();
    List<BulkFileDto> findByStatus(Context context , String status);
    BulkUploadRequestResponseDTO getFile(UUID uuid);
    List<BulkFileDto> getPooledTasksForReviewer(Context context, UUID reviewerId);
}




// dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/service/impl/BulkUploadRequestServiceImpl.java


package org.dspace.app.rest.diracai.service.impl;


import org.dspace.app.rest.converter.DSpaceRunnableParameterConverter;
import org.dspace.app.rest.diracai.Repository.BulkUploadItemRepository;
import org.dspace.app.rest.diracai.Repository.BulkUploadRequestRepository;
import org.dspace.app.rest.diracai.dto.*;
import org.dspace.app.rest.diracai.service.BulkUploadRequestService;
import org.dspace.app.rest.diracai.util.BulkUploadRequestUtil;
import org.dspace.app.rest.diracai.util.FileMultipartFile;
import org.dspace.app.rest.repository.ScriptRestRepository;
import org.dspace.authorize.service.AuthorizeService;
import org.dspace.content.Diracai.BulkUploadItem;
import org.dspace.content.Diracai.BulkUploadItemMetadata;
import org.dspace.content.Diracai.BulkUploadRequest;
import org.dspace.content.service.*;
import org.dspace.core.Context;
import org.dspace.eperson.EPerson;
import org.dspace.content.Collection;

import org.dspace.eperson.Group;
import org.dspace.eperson.service.EPersonService;
import org.dspace.eperson.service.GroupService;
import org.dspace.scripts.service.ScriptService;
import org.dspace.services.RequestService;
import org.dspace.xoai.services.api.context.ContextService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;
import org.dspace.services.ConfigurationService;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.io.FileNotFoundException;

import java.sql.SQLException;
import java.util.*;
import java.util.stream.Collectors;



@Service
public class BulkUploadRequestServiceImpl implements BulkUploadRequestService {

    @Autowired
    private ScriptService scriptService;

    @Autowired
    private ConfigurationService configurationService;

    @Autowired
    private BulkUploadRequestRepository bulkUploadRequestRepository;

    @Autowired
    private BulkUploadItemRepository bulkUploadItemRepository;

    @Autowired
    private ItemService itemService;

    @Autowired
    private ContextService contextService;

    @Autowired
    private BulkUploadRequestUtil bulkUploadRequestUtil;

    @Autowired
    private CollectionService collectionService;

    @Autowired
    private WorkspaceItemService workspaceItemService;

    @Autowired
    private InstallItemService installItemService;

    @Autowired
    private BitstreamService bitstreamService;

    @Autowired
    private BundleService bundleService;

    @Autowired
    private DSpaceRunnableParameterConverter parameterConverter;

    @Autowired
    private ScriptRestRepository scriptRestRepository;

    @Autowired
    private RequestService requestService;


    @Autowired
    private EPersonService epersonService;

    @Autowired
    private AuthorizeService authorizeService;

    @Autowired
    private GroupService groupService;


    private static final Logger logger = LoggerFactory.getLogger(BulkUploadRequestServiceImpl.class);

    private static final Logger log = LoggerFactory.getLogger(BulkUploadRequestServiceImpl.class);



    @Override
    public List<BulkFileDto> getPooledTasksForReviewer(Context context, UUID reviewerId) {
        List<BulkUploadRequest> bulkUploadRequests = new ArrayList<>();
        List<BulkFileDto> bulkFileDtos = new ArrayList<>();

        EPerson user;
        try {
            user = epersonService.find(context, reviewerId);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }

        if (user == null) {
            throw new RuntimeException("User not found: " + reviewerId);
        }

        boolean isAdmin;
        try {
            isAdmin = authorizeService.isAdmin(context, user);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }

        if (isAdmin) {
            bulkUploadRequests.addAll(bulkUploadRequestRepository.findAllByStatusNot("CLAIMED"));
        } else {
            bulkUploadRequests.addAll(bulkUploadRequestRepository.findByReviewerId(reviewerId));
            bulkUploadRequests.addAll(bulkUploadRequestRepository.findByUploaderId(reviewerId));
        }

        for (BulkUploadRequest bulkUploadRequest : bulkUploadRequests) {
            BulkFileDto bulkFileDto = new BulkFileDto();

            // Prepare collection info
            BulkFileCollectionDto bulkFileCollectionDto = new BulkFileCollectionDto();
            try {
                Collection collection = collectionService.find(context, bulkUploadRequest.getCollectionId());
                if (collection != null) {
                    bulkFileCollectionDto.setCollectionId(collection.getID());
                    bulkFileCollectionDto.setCollectionName(collection.getName());
                }
            } catch (SQLException e) {
                throw new RuntimeException("Error fetching collection", e);
            }

            // Prepare uploader info
            BulkFileUser uploader = new BulkFileUser();
            try {
                EPerson uploaderPerson = epersonService.find(context, bulkUploadRequest.getUploaderId());
                if (uploaderPerson != null) {
                    uploader.setUuid(uploaderPerson.getID());
                    uploader.setUserName(uploaderPerson.getName());
                    uploader.setDate(bulkUploadRequest.getUploadedDate());
                }
            } catch (SQLException e) {
                throw new RuntimeException("Error fetching uploader", e);
            }

            // Prepare reviewer info
            BulkFileUser reviewer = new BulkFileUser();
            try {
                EPerson reviewerPerson = epersonService.find(context, bulkUploadRequest.getReviewerId());
                if (reviewerPerson != null) {
                    reviewer.setUuid(reviewerPerson.getID());
                    reviewer.setUserName(reviewerPerson.getName());
                    reviewer.setDate(bulkUploadRequest.getReviewedDate());
                }
            } catch (SQLException e) {
                throw new RuntimeException("Error fetching reviewer", e);
            }

            // Assemble the DTO
            bulkFileDto.setBulkFileId(bulkUploadRequest.getBulkUploadId());
            bulkFileDto.setCollection(bulkFileCollectionDto);
            bulkFileDto.setFileName(bulkUploadRequest.getFilename());
            bulkFileDto.setStatus(bulkUploadRequest.getStatus());
            bulkFileDto.setUploader(uploader);
            bulkFileDto.setReviewer(reviewer);

            bulkFileDtos.add(bulkFileDto);
        }

        return bulkFileDtos;
    }





    @Override
    public BulkUploadRequest approveRequest(UUID uuid, Context context, AuthTokenPayload auth) {
        log.info("✅ Approving bulk upload request ID: {}", uuid);

        BulkUploadRequest req = bulkUploadRequestRepository.findById(uuid)
                .orElseThrow(() -> new RuntimeException("BulkUploadRequest not found: " + uuid));

        EPerson currentUser = context.getCurrentUser();
        if (currentUser == null) {
            throw new RuntimeException("No current user found in context");
        }

        boolean isAdmin;
        try {
            isAdmin = authorizeService.isAdmin(context);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }

        Group reviewerGroup;
        try {
            reviewerGroup = groupService.findByName(context, "reviewer");
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }

        boolean isReviewer;
        try {
            isReviewer = reviewerGroup != null && groupService.isMember(context, currentUser, reviewerGroup);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }

        if (!(isAdmin && isReviewer)) {
            log.warn("⛔ Access denied: User {} must be both Admin and Reviewer.", currentUser.getID());
            throw new RuntimeException("User must be both Admin and Reviewer to approve the request.");
        }

        req.setStatus("APPROVED");
        req.setReviewerId(currentUser.getID());
        req.setReviewedDate(new Date());
        bulkUploadRequestRepository.save(req);

        try {
            String dspaceDir = configurationService.getProperty("dspace.dir");
            File zipFile = new File(dspaceDir + "/bulk_upload/" + uuid + "/" + uuid + ".zip");

            if (!zipFile.exists()) {
                throw new FileNotFoundException("ZIP file not found: " + zipFile.getAbsolutePath());
            }

            MultipartFile multipartFile = new FileMultipartFile(zipFile);

            scriptRestRepository.startProcess(context, "import", List.of(multipartFile));
            log.info("✅ Import script triggered via startProcess for UUID: {}", uuid);

        } catch (Exception e) {
            log.error("❌ Script trigger failed for UUID {}: {}", uuid, e.getMessage(), e);
            throw new RuntimeException("Script trigger failed", e);
        }

        return req;
    }



    @Override
    public BulkUploadRequest rejectRequest(UUID uuid, Context context) {
        BulkUploadRequest req = bulkUploadRequestRepository.findById(uuid).orElseThrow();


        EPerson currentUser = context.getCurrentUser();
        if (currentUser == null) {
            throw new RuntimeException("No current user found in context");
        }

        boolean isAdmin;
        try {
            isAdmin = authorizeService.isAdmin(context);
        } catch (SQLException e) {
            throw new RuntimeException("Error checking admin role", e);
        }

        Group reviewerGroup;
        try {
            reviewerGroup = groupService.findByName(context, "reviewer");
            if (reviewerGroup == null) {
                throw new RuntimeException("Reviewer group not found");
            }
        } catch (SQLException e) {
            throw new RuntimeException("Error retrieving Reviewer group", e);
        }

        boolean isReviewer;
        try {
            isReviewer = groupService.isMember(context, currentUser, reviewerGroup);
        } catch (SQLException e) {
            throw new RuntimeException("Error checking Reviewer group membership", e);
        }

        // ✅ Enforce both Admin AND Reviewer requirement
        if (!(isAdmin && isReviewer)) {
            log.warn("⛔ Access denied: User {} must be both Admin and Reviewer to reject.", currentUser.getID());
            throw new RuntimeException("User must be both Admin and Reviewer to reject the request.");
        }
        req.setStatus("REJECTED");
        req.setReviewerId(currentUser.getID());
        req.setReviewedDate(new Date());
        return bulkUploadRequestRepository.save(req);
    }


    @Override
    public BulkUploadRequest createRequest(Context context, MultipartFile file, UUID uuid) {
        EPerson currentUser = context.getCurrentUser();
        if (currentUser == null) {
            log.error("❌ No authenticated user in context during bulk upload creation.");
            throw new RuntimeException("No authenticated user in context");
        }

        // Check if user is Admin or in Uploader group
        boolean isAdmin = false;
        try {
            isAdmin = authorizeService.isAdmin(context);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
        Group uploaderGroup;
        try {
            uploaderGroup = groupService.findByName(context, "uploader");
        } catch (Exception e) {
            log.error("❌ Could not find Uploader group.", e);
            throw new RuntimeException("Uploader group not found", e);
        }

        boolean isUploader = false;
        try {
            isUploader = uploaderGroup != null && groupService.isMember(context, currentUser, uploaderGroup);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }

        if (!(isAdmin && isUploader)) {
            log.warn("⛔ Access denied: User {} must be both Admin and Uploader.", currentUser.getID());
            throw new RuntimeException("User must be both Admin and Uploader to upload files.");
        }
        log.info("📥 Initiating bulk upload request creation by user: {}", currentUser.getID());
        log.info("📎 Received file: name='{}', size={} bytes", file.getOriginalFilename(), file.getSize());

        BulkUploadRequest req = new BulkUploadRequest();
        req.setUploaderId(currentUser.getID());
        req.setFilename(file.getOriginalFilename());
        req.setStatus("CLAIMED");
        req.setUploadedDate(new Date());
        req.setCollectionId(uuid);

        req = bulkUploadRequestRepository.save(req);
        log.info("📝 Created BulkUploadRequest with ID: {} for collection: {}", req.getBulkUploadId(), uuid);

        try {
            bulkUploadRequestUtil.handleZipUpload(context, file, req.getBulkUploadId());
            log.info("✅ Successfully extracted and processed ZIP for request ID: {}", req.getBulkUploadId());
        } catch (Exception e) {
            log.error("❌ Failed to extract ZIP file for request ID: {} — {}", req.getBulkUploadId(), e.getMessage(), e);
            throw new RuntimeException("Failed to extract zip file", e);
        }

        return req;
    }




    @Override
    public List<BulkUploadRequest> findAll() {
        return bulkUploadRequestRepository.findAll();
    }

    @Override
    public List<BulkFileDto> findByStatus(Context context , String status) {
        log.info("🔍 Fetching BulkUploadRequests with status: {}", status);

        List<BulkUploadRequest> bulkUploadRequests = bulkUploadRequestRepository.findByStatus(status);
        log.info("📄 Found {} requests with status '{}'", bulkUploadRequests.size(), status);

        List<BulkFileDto> bulkFileDtos = new ArrayList<>();

        for (BulkUploadRequest bulkUploadRequest : bulkUploadRequests) {
            log.debug("📦 Processing BulkUploadRequest ID: {}", bulkUploadRequest.getBulkUploadId());

            BulkFileDto bulkFileDto = new BulkFileDto();
            BulkFileCollectionDto bulkFileCollectionDto = new BulkFileCollectionDto();
            BulkFileUser uploaderUser = new BulkFileUser();
            BulkFileUser reviewerUser = new BulkFileUser();

            try {
                Collection collection = collectionService.find(context, bulkUploadRequest.getCollectionId());
                bulkFileCollectionDto.setCollectionId(bulkUploadRequest.getCollectionId());
                bulkFileCollectionDto.setCollectionName(collection != null ? collection.getName() : "Unknown");
                log.debug("📁 Collection resolved: {}", bulkFileCollectionDto);
            } catch (SQLException e) {
                log.error("❌ Failed to fetch collection for ID: {}", bulkUploadRequest.getCollectionId(), e);
                throw new RuntimeException(e);
            }

            try {
                EPerson uploader = epersonService.find(context, bulkUploadRequest.getUploaderId());
                if (uploader != null) {
                    uploaderUser.setUuid(uploader.getID());
                    uploaderUser.setUserName(uploader.getName());
                    uploaderUser.setDate(bulkUploadRequest.getUploadedDate());
                    log.debug("👤 Uploader info: {}", uploaderUser);
                }
            } catch (SQLException e) {
                log.error("❌ Failed to fetch uploader for ID: {}", bulkUploadRequest.getUploaderId(), e);
                throw new RuntimeException(e);
            }

            try {
                EPerson reviewer = epersonService.find(context, bulkUploadRequest.getReviewerId());
                if (reviewer != null) {
                    reviewerUser.setUuid(reviewer.getID());
                    reviewerUser.setUserName(reviewer.getName());
                    reviewerUser.setDate(bulkUploadRequest.getReviewedDate());
                    log.debug("👤 Reviewer info: {}", reviewerUser);
                }
            } catch (SQLException e) {
                log.error("❌ Failed to fetch reviewer for ID: {}", bulkUploadRequest.getReviewerId(), e);
                throw new RuntimeException(e);
            }

            bulkFileDto.setBulkFileId(bulkUploadRequest.getBulkUploadId());
            bulkFileDto.setCollection(bulkFileCollectionDto);
            bulkFileDto.setFileName(bulkUploadRequest.getFilename());
            bulkFileDto.setStatus(bulkUploadRequest.getStatus());
            bulkFileDto.setUploader(uploaderUser);
            bulkFileDto.setReviewer(reviewerUser);

            bulkFileDtos.add(bulkFileDto);
        }

        log.info("✅ Prepared {} BulkFileDto objects for response", bulkFileDtos.size());
        return bulkFileDtos;
    }



    @Override
    public BulkUploadRequestResponseDTO getFile(UUID uuid) {
        logger.info("Starting to process bulk upload file request for UUID: {}", uuid);

        logger.debug("Attempting to find bulk upload request in repository");
        BulkUploadRequest request = bulkUploadRequestRepository.findById(uuid)
                .orElseThrow(() -> {
                    logger.error("Bulk upload request not found for UUID: {}", uuid);
                    return new RuntimeException("Request not found: " + uuid);
                });
        logger.info("Found bulk upload request - ID: {}, Filename: {}, Status: {}",
                request.getBulkUploadId(), request.getFilename(), request.getStatus());

        logger.debug("Querying for associated items");
        List<BulkUploadItem> items = bulkUploadItemRepository.findWithMetadataByUploadRequest(uuid);
        logger.info("Found {} items associated with request UUID: {}", items.size(), uuid);

        List<BulkUploadItemDTO> itemDTOs = items.stream().map(item -> {
            BulkUploadItemDTO dto = new BulkUploadItemDTO();
            dto.setItemId(item.getUuid());
            dto.setItemFolder(item.getItemFolder());
            Map<String, String> metadataMap = item.getMetadata().stream()
                    .collect(Collectors.toMap(
                            BulkUploadItemMetadata::getKey,
                            BulkUploadItemMetadata::getValue,
                            (oldValue, newValue) -> newValue
                    ));
            dto.setMetadata(metadataMap);

            return dto;
        }).toList();


        BulkUploadRequestResponseDTO response = new BulkUploadRequestResponseDTO();
        response.setRequestId(request.getBulkUploadId());
        response.setFilename(request.getFilename());
        response.setStatus(request.getStatus());
        response.setUploaderId(request.getUploaderId());
        response.setUploadedDate(request.getUploadedDate());
        response.setItems(itemDTOs);

        return response;
    }

}




// dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/util/BulkUploadRequestUtil.java


package org.dspace.app.rest.diracai.util;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.io.FileUtils;
import org.dspace.app.rest.diracai.Repository.BulkUploadItemRepository;
import org.dspace.app.rest.diracai.Repository.BulkUploadRequestRepository;
import org.dspace.app.rest.diracai.service.BulkUploadItemService;
import org.dspace.app.rest.diracai.service.impl.BulkUploadRequestServiceImpl;
import org.dspace.content.Diracai.BulkUploadItem;
import org.dspace.content.Diracai.BulkUploadItemMetadata;
import org.dspace.content.Diracai.BulkUploadRequest;
import org.dspace.core.Context;
import org.dspace.services.ConfigurationService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.NodeList;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.StandardCopyOption;
import java.sql.SQLException;
import java.util.*;

@Component
public class BulkUploadRequestUtil {

    @Autowired
    private ConfigurationService configurationService;

    @Autowired
    private BulkUploadRequestRepository bulkUploadRequestRepository;

    @Autowired
    private BulkUploadItemService bulkUploadItemService;

    @Autowired
    private BulkUploadItemRepository bulkUploadItemRepository;


    private static final Logger logger = LoggerFactory.getLogger(BulkUploadRequestUtil.class);

    public BulkUploadRequest handleZipUpload(Context context, MultipartFile file, UUID uuid) throws IOException, SQLException {
        logger.info("Starting ZIP upload handling for BulkUploadRequest UUID: {}", uuid);

        // Step 1: Save uploaded ZIP file to temp
        File tempZip = File.createTempFile("bulk-upload-", ".zip");
        logger.debug("Created temporary ZIP file at {}", tempZip.getAbsolutePath());
        file.transferTo(tempZip);
        logger.info("Transferred uploaded file to temporary location.");

        // Step 2: Define upload folder path
        String dspaceDir = configurationService.getProperty("dspace.dir");
        String uploadFolderPath = dspaceDir + File.separator + "bulk_upload" + File.separator + uuid;
        File uploadFolder = new File(uploadFolderPath);

        if (!uploadFolder.exists() && !uploadFolder.mkdirs()) {
            logger.error("Failed to create upload folder: {}", uploadFolderPath);
            throw new IOException("Could not create upload directory.");
        }
        logger.info("Upload folder created at: {}", uploadFolderPath);

        Files.copy(tempZip.toPath(), new File(uploadFolder, uuid.toString() + ".zip").toPath(), StandardCopyOption.REPLACE_EXISTING);


        // Step 3: Unzip the file
        ZipUtils.unzip(tempZip, uploadFolder);
        logger.info("Unzipped content into upload folder.");

        // Step 4: Get BulkUploadRequest
        Optional<BulkUploadRequest> optionalRequest = bulkUploadRequestRepository.findById(uuid);
        if (!optionalRequest.isPresent()) {
            logger.error("BulkUploadRequest not found for UUID: {}", uuid);
            throw new RuntimeException("BulkUploadRequest not found for UUID: " + uuid);
        }
        BulkUploadRequest request = optionalRequest.get();
        logger.info("Fetched BulkUploadRequest with ID: {}", request.getBulkUploadId());

        // Step 5: Process each item directory
        File[] itemDirs = uploadFolder.listFiles(File::isDirectory);
        if (itemDirs != null) {
            logger.info("Found {} item folders to process.", itemDirs.length);
            for (File itemDir : itemDirs) {
                logger.debug("Processing folder: {}", itemDir.getName());

                File dublinCore = new File(itemDir, "dublin_core.xml");
                if (!dublinCore.exists()) {
                    logger.warn("Missing dublin_core.xml in folder: {}", itemDir.getAbsolutePath());
                    continue;
                }

                // Create BulkUploadItem
                BulkUploadItem item = new BulkUploadItem();
                item.setUploadRequest(request.getBulkUploadId());
                item.setItemFolder(itemDir.getName());

                item = bulkUploadItemService.create(context, item);
                logger.info("Created BulkUploadItem for folder: {}, item ID: {}", itemDir.getName(), item.getUuid());

                // Extract and apply metadata
                try {
                    parseDublinCoreXML(dublinCore, item);
                    logger.debug("Parsed and applied Dublin Core metadata for item: {}", item.getUuid());
                } catch (Exception e) {
                    logger.error("Failed to parse Dublin Core for item: {} - {}", item.getUuid(), e.getMessage(), e);
                }
            }
        } else {
            logger.warn("No item folders found in uploaded ZIP.");
        }

        // Step 6: Clean up temporary file
        if (tempZip.exists() && !tempZip.delete()) {
            logger.warn("Failed to delete temporary zip file: {}", tempZip.getAbsolutePath());
        } else {
            logger.debug("Deleted temporary zip file: {}", tempZip.getAbsolutePath());
        }

        logger.info("Completed ZIP upload handling for BulkUploadRequest UUID: {}", uuid);
        return request;
    }



    private void parseDublinCoreXML(File xmlFile, BulkUploadItem item) {
        logger.debug("Metadata extractor called for item: {}", item.getUuid());

        List<BulkUploadItemMetadata> metadataList = new ArrayList<>();

        try {
            DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
            DocumentBuilder builder = factory.newDocumentBuilder();
            Document doc = builder.parse(xmlFile);

            NodeList nodes = doc.getElementsByTagName("dcvalue");

            for (int i = 0; i < nodes.getLength(); i++) {
                Element el = (Element) nodes.item(i);
                String element = el.getAttribute("element");
                String qualifier = el.getAttribute("qualifier");

                String key = "dc." + element;
                if (qualifier != null && !qualifier.isEmpty() && !"none".equalsIgnoreCase(qualifier)) {
                    key += "." + qualifier;
                }

                String value = el.getTextContent().trim();

                logger.debug("Extracted metadata key: {}, value: {}", key, value);

                BulkUploadItemMetadata meta = new BulkUploadItemMetadata();
                meta.setKey(key);
                meta.setValue(value);
                meta.setItem(item);
                metadataList.add(meta);
            }

            item.getMetadata().clear();
            item.getMetadata().addAll(metadataList);
            bulkUploadItemRepository.save(item);

        } catch (Exception e) {
            logger.error("Error parsing Dublin Core XML for item {}: {}", item.getUuid(), e.getMessage(), e);
        }
    }

}



//dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/Repository/BulkUploadRequestRepository.java


package org.dspace.app.rest.diracai.Repository;

import org.dspace.content.Diracai.BulkUploadRequest;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;
import java.util.Optional;
import java.util.UUID;

public interface BulkUploadRequestRepository extends JpaRepository<BulkUploadRequest, UUID> {
    List<BulkUploadRequest> findByStatus(String status);
    List<BulkUploadRequest> findAllByStatusNot(String status);
    List<BulkUploadRequest> findByReviewerId(UUID reviewerId);
    List<BulkUploadRequest> findByUploaderId(UUID uploaderId);

}




//dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/Repository/BulkUploadItemRepository.java




package org.dspace.app.rest.diracai.Repository;


import org.dspace.content.Diracai.BulkUploadItem;
import org.dspace.content.Diracai.BulkUploadRequest;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.util.List;
import java.util.UUID;

public interface BulkUploadItemRepository extends JpaRepository<BulkUploadItem, UUID> {

    @Query("SELECT i FROM BulkUploadItem i LEFT JOIN FETCH i.metadata WHERE i.uploadRequest = :uploadRequestId")
    List<BulkUploadItem> findWithMetadataByUploadRequest(@Param("uploadRequestId") UUID uploadRequestId);

}


// dspace_backend_latest-main/dspace-api/src/main/java/org/dspace/content/Diracai/BulkUploadItem.java

package org.dspace.content.Diracai;

import jakarta.persistence.*;
import org.hibernate.annotations.GenericGenerator;

import java.util.*;

@Entity
@Table(name = "bulk_upload_item")
public class BulkUploadItem {

    @Id
    @GeneratedValue
    @GenericGenerator(name = "uuid", strategy = "uuid2")
    @Column(name = "bulk_upload_id", columnDefinition = "uuid", updatable = false, nullable = false)
    private UUID uuid;

    @Column(name = "upload_request_id", nullable = false, columnDefinition = "uuid")
    private UUID uploadRequest;

    @Column(name = "item_folder", nullable = false)
    private String itemFolder;

    @OneToMany(mappedBy = "item", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<BulkUploadItemMetadata> metadata = new ArrayList<>();

    // Getters and setters
    public UUID getUuid() { return uuid; }
    public void setUuid(UUID uuid) { this.uuid = uuid; }

    public UUID getUploadRequest() { return uploadRequest; }
    public void setUploadRequest(UUID uploadRequest) { this.uploadRequest = uploadRequest; }

    public String getItemFolder() { return itemFolder; }
    public void setItemFolder(String itemFolder) { this.itemFolder = itemFolder; }

    public List<BulkUploadItemMetadata> getMetadata() { return metadata; }
    public void setMetadata(List<BulkUploadItemMetadata> metadata) { this.metadata = metadata; }
}



// dspace_backend_latest-main/dspace-api/src/main/java/org/dspace/content/Diracai/BulkUploadRequest.java



package org.dspace.content.Diracai;

import jakarta.persistence.*;
import org.hibernate.annotations.GenericGenerator;

import java.util.Date;
import java.util.UUID;

@Entity
@Table(name = "bulk_upload_request")
public class BulkUploadRequest {

    @Id
    @GeneratedValue
    @GenericGenerator(name = "uuid", strategy = "uuid2")
    @Column(name = "bulk_upload_id", columnDefinition = "uuid", updatable = false, nullable = false)
    private UUID bulkUploadId;

    @Column(nullable = false)
    private UUID uploaderId;

    @Column(nullable = false)
    private UUID collectionId;


    @Column(nullable = false)
    private UUID reviewerId;

    @Column(nullable = false)
    private String filename;

    @Column(nullable = false)
    private String status; // CLAIMED, APPROVED, REJECTED, PENDING

    @Column(nullable = false)
    private Date uploadedDate;

    @Column(nullable = false)
    private Date reviewedDate;

    public Date getReviewedDate() {
        return reviewedDate;
    }

    public void setReviewedDate(Date reviewedDate) {
        this.reviewedDate = reviewedDate;
    }

    public UUID getBulkUploadId() {
        return bulkUploadId;
    }

    public void setBulkUploadId(UUID bulkUploadId) {
        this.bulkUploadId = bulkUploadId;
    }

    public UUID getUploaderId() {
        return uploaderId;
    }

    public void setUploaderId(UUID uploaderId) {
        this.uploaderId = uploaderId;
    }

    public String getFilename() {
        return filename;
    }

    public void setFilename(String filename) {
        this.filename = filename;
    }

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }

    public Date getUploadedDate() {
        return uploadedDate;
    }

    public void setUploadedDate(Date uploadedDate) {
        this.uploadedDate = uploadedDate;
    }

    public UUID getReviewerId() {
        return reviewerId;
    }

    public void setReviewerId(UUID reviewerId) {
        this.reviewerId = reviewerId;
    }

    public UUID getCollectionId() {
        return collectionId;
    }

    public void setCollectionId(UUID collectionId) {
        this.collectionId = collectionId;
    }
}




// dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/dto/BulkFileCollectionDto.java



package org.dspace.app.rest.diracai.dto;

import java.util.UUID;

public class BulkFileCollectionDto {
    private UUID collectionId;
    private String collectionName;

    public void setCollectionId(UUID collectionId) {
        this.collectionId = collectionId;
    }

    public UUID getCollectionId() {
        return collectionId;
    }

    public String getCollectionName() {
        return collectionName;
    }

    public void setCollectionName(String collectionName) {
        this.collectionName = collectionName;
    }

    @Override
    public String toString() {
        return "BulkFileCollectionDto{" +
                "collectionId=" + collectionId +
                ", collectionName='" + collectionName + '\'' +
                '}';
    }

}



// dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/dto/BulkFileDto.java

package org.dspace.app.rest.diracai.dto;

import java.util.UUID;

public class BulkFileDto {
    private UUID BulkFileId;
    private BulkFileCollectionDto collection;
    private BulkFileUser uploader;
    private String status;
    private String fileName;
    private BulkFileUser reviewer;

    public BulkFileUser getReviewer() {
        return reviewer;
    }

    public BulkFileUser getUploader() {
        return uploader;
    }

    public void setReviewer(BulkFileUser reviewer) {
        this.reviewer = reviewer;
    }

    public void setUploader(BulkFileUser uploader) {
        this.uploader = uploader;
    }

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }

    public String getFileName() {
        return fileName;
    }

    public void setFileName(String fileName) {
        this.fileName = fileName;
    }

    public UUID getBulkFileId() {
        return BulkFileId;
    }

    public void setBulkFileId(UUID bulkFileId) {
        BulkFileId = bulkFileId;
    }

    public BulkFileCollectionDto getCollection() {
        return collection;
    }

    public void setCollection(BulkFileCollectionDto collection) {
        this.collection = collection;
    }

}


// dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/dto/BulkFileUser.java



package org.dspace.app.rest.diracai.dto;

import java.util.Date;
import java.util.UUID;

public class BulkFileUser {
    private UUID uuid;
    private String userName;
    private Date date;

    public void setUuid(UUID uuid) {
        this.uuid = uuid;
    }

    public UUID getUuid() {
        return uuid;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public Date getDate() {
        return date;
    }

    public void setDate(Date date) {
        this.date = date;
    }

    @Override
    public String toString() {
        return "BulkFileUser{" +
                "uuid=" + uuid +
                ", userName='" + userName + '\'' +
                ", date=" + date +
                '}';
    }
}


// dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/dto/BulkUploadItemDTO.java


package org.dspace.app.rest.diracai.dto;

import java.util.Map;
import java.util.UUID;

public class BulkUploadItemDTO {
    private UUID itemId;
    private String itemFolder;
    private Map<String, String> metadata;

    public void setItemFolder(String itemFolder) {
        this.itemFolder = itemFolder;
    }

    public String getItemFolder() {
        return itemFolder;
    }

    public UUID getItemId() {
        return itemId;
    }

    public void setItemId(UUID itemId) {
        this.itemId = itemId;
    }

    public Map<String, String> getMetadata() {
        return metadata;
    }

    public void setMetadata(Map<String, String> metadata) {
        this.metadata = metadata;
    }
}
