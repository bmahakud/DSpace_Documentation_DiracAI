package org.dspace.app.rest.diracai.controller;

import jakarta.servlet.http.HttpServletRequest;
import org.dspace.app.rest.diracai.dto.Request.BitstreamCommentRequest;
import org.dspace.app.rest.diracai.dto.Response.BitstreamCommentResponse;
import org.dspace.app.rest.diracai.service.BitstreamCommentService;
import org.dspace.app.rest.utils.ContextUtil;
import org.dspace.core.Context;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.UUID;

@RestController
@RequestMapping("/api/bitstream/comment")
public class BitstreamCommentController {

    @Autowired
    private BitstreamCommentService service;

    @PostMapping
    public BitstreamCommentResponse create(@RequestBody BitstreamCommentRequest request, HttpServletRequest httpRequest) {
        Context context = ContextUtil.obtainContext(httpRequest);
        return service.create(context, request);
    }

    @PutMapping("/{id}")
    public BitstreamCommentResponse update(@PathVariable int id, @RequestBody BitstreamCommentRequest request, HttpServletRequest httpRequest) {
        Context context = ContextUtil.obtainContext(httpRequest);
        return service.update(context, id, request);
    }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable int id) {
        service.delete(id);
    }

    @GetMapping("/{id}")
    public BitstreamCommentResponse getById(@PathVariable int id,HttpServletRequest httpRequest) {
        Context context = ContextUtil.obtainContext(httpRequest);
        return service.getById(context,id);
    }

    @GetMapping("/bitstream/{bitstreamId}")
    public List<BitstreamCommentResponse> getByBitstream(@PathVariable UUID bitstreamId,HttpServletRequest httpRequest) {
        Context context = ContextUtil.obtainContext(httpRequest);
        return service.getByBitstreamId(context,bitstreamId);
    }
}


package org.dspace.app.rest.diracai.service;

import org.dspace.app.rest.diracai.dto.Request.BitstreamCommentRequest;
import org.dspace.app.rest.diracai.dto.Response.BitstreamCommentResponse;
import org.dspace.core.Context;

import java.util.List;
import java.util.UUID;

public interface BitstreamCommentService {
    BitstreamCommentResponse create(Context context, BitstreamCommentRequest request);
    BitstreamCommentResponse update(Context context, int id, BitstreamCommentRequest request);
    void delete(int id);
    BitstreamCommentResponse getById(Context context,int id);
    List<BitstreamCommentResponse> getByBitstreamId(Context context,UUID bitstreamId);
}



package org.dspace.app.rest.diracai.service.impl;

import org.dspace.app.rest.diracai.dto.Request.BitstreamCommentRequest;
import org.dspace.app.rest.diracai.dto.Response.BitstreamCommentResponse;
import org.dspace.app.rest.diracai.Repository.BitstreamCommentRepository;
import org.dspace.app.rest.diracai.service.BitstreamCommentService;
import org.dspace.authorize.service.AuthorizeService;
import org.dspace.content.Diracai.BitstreamComment;
import org.dspace.core.Context;
import org.dspace.eperson.EPerson;
import org.dspace.eperson.service.EPersonService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.sql.SQLException;
import java.util.List;
import java.util.UUID;
import java.util.stream.Collectors;

@Service
public class BitstreamCommentServiceImpl implements BitstreamCommentService {

    @Autowired
    private BitstreamCommentRepository repository;

    @Autowired
    private AuthorizeService authorizeService;

    @Autowired
    private EPersonService ePersonService;


    @Override
    public BitstreamCommentResponse create(Context context, BitstreamCommentRequest request) {
        EPerson user = context.getCurrentUser();

        try {
            if (!authorizeService.isAdmin(context,user)) {
                throw new RuntimeException("Only admin users are allowed to create comments.");
            }
        } catch (SQLException e) {
            throw new RuntimeException("Authorization check failed", e);
        }

        BitstreamComment comment = new BitstreamComment(
                request.getComment(),
                request.getBitstreamId(),
                user.getID()
        );

        BitstreamComment saved = repository.save(comment);

        return mapToResponse(context , saved);
    }

    @Override
    public BitstreamCommentResponse update(Context context, int id, BitstreamCommentRequest request) {
        BitstreamComment existing = repository.findById(id)
                .orElseThrow(() -> new RuntimeException("Comment not found with id: " + id));
        existing.setText(request.getComment());
        existing.setBitstreamId(request.getBitstreamId());

        BitstreamComment saved = repository.save(existing);
        return mapToResponse(context,saved);
    }

    @Override
    public void delete(int id) {
        BitstreamComment comment = repository.findById(id)
                .orElseThrow(() -> new RuntimeException("Comment not found with id: " + id));
        comment.setDeleted(true);
        repository.save(comment);
    }

    @Override
    public BitstreamCommentResponse getById(Context context , int id) {
        BitstreamComment comment = repository.findById(id)
                .orElseThrow(() -> new RuntimeException("Comment not found with id: " + id));
        return mapToResponse(context,comment);
    }

//    @Override
//    public List<BitstreamCommentResponse> getByBitstreamId(UUID bitstreamId) {
//        return repository.findByBitstreamId(bitstreamId)
//                .stream()
//                .map(this::mapToResponse)
//                .collect(Collectors.toList());
//    }
//
//    private BitstreamCommentResponse mapToResponse(BitstreamComment comment) {
//        EPerson user = ePersonService.find(context, comment.getCommenterId());
//        String commenterName = (user != null) ? user.getFullName() : "Unknown User";
//
//        return new BitstreamCommentResponse(
//                comment.getId(),
//                comment.getCommentDate(),
//                comment.getText(),
//                comment.isDeleted(),
//
//        );
//    }

    @Override
    public List<BitstreamCommentResponse> getByBitstreamId(Context context, UUID bitstreamId) {
        return repository.findByBitstreamIdAndIsDeletedFalse(bitstreamId)
                .stream()
                .map(comment -> mapToResponse(context, comment))
                .collect(Collectors.toList());
    }

    private BitstreamCommentResponse mapToResponse(Context context, BitstreamComment comment) {
        EPerson user = null;
        try {
            user = ePersonService.find(context, comment.getCommenterId());
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
        String commenterName = (user != null) ? user.getFullName() : "Unknown User";

        return new BitstreamCommentResponse(
                comment.getId(),
                comment.getCommentDate(),
                comment.getText(),
                comment.isDeleted(),
                commenterName
        );
    }

}


package org.dspace.app.rest.diracai.Repository;

import org.dspace.content.Diracai.BitstreamComment;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.UUID;

@Repository
public interface BitstreamCommentRepository extends JpaRepository<BitstreamComment, Integer> {
    List<BitstreamComment> findByBitstreamIdAndIsDeletedFalse(UUID bitstreamId);
}



package org.dspace.content.Diracai;

import jakarta.persistence.*;
import java.time.LocalDateTime;
import java.util.UUID;

@Entity
@Table(name = "bitstream_comment")
public class BitstreamComment {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column(nullable = false)
    private String text;

    @Column(name = "bitstream_id", nullable = false)
    private UUID bitstreamId;

    @Column(name = "commenter_id", nullable = false)
    private UUID commenterId;

    @Column(name = "comment_date", nullable = false)
    private LocalDateTime commentDate;

    @Column(name = "is_deleted", nullable = false)
    private boolean isDeleted;

    // Default constructor (required by JPA)
    public BitstreamComment() {}

    // Custom constructor for convenience
    public BitstreamComment(String text, UUID bitstreamId, UUID commenterId) {
        this.text = text;
        this.bitstreamId = bitstreamId;
        this.commenterId = commenterId;
        this.isDeleted = false; // Default to false
    }

    @PrePersist
    protected void onCreate() {
        this.commentDate = LocalDateTime.now();
    }

    // Getters and Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getText() { return text; }
    public void setText(String text) { this.text = text; }

    public UUID getBitstreamId() { return bitstreamId; }
    public void setBitstreamId(UUID bitstreamId) { this.bitstreamId = bitstreamId; }

    public UUID getCommenterId() { return commenterId; }
    public void setCommenterId(UUID commenterId) { this.commenterId = commenterId; }

    public LocalDateTime getCommentDate() { return commentDate; }
    public void setCommentDate(LocalDateTime commentDate) { this.commentDate = commentDate; }

    public boolean isDeleted() { return isDeleted; }
    public void setDeleted(boolean deleted) { isDeleted = deleted; }
}



